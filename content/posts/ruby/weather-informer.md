---
title: "Exploring Ruby: Building a Chatbot"
date: 2023-06-14
tags: ["ruby", "chatbot", "Ruby on Rails", "sidekiq", "telegram"]
author: "benetis"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Building weather information downloader and letting chatbot user interact with data"
disableHLJS: false # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/benetis/benetis.me/blob/master/content/posts/ruby/weather-informer.md"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

## Introduction

New Ruby community member here. Looking to get familiar with language and share my insights.

Recently, I've accepted an offer to work in a different company. They use Ruby as their main language. 
I've never worked with Ruby in a professional setting, but I did already take a stab at Ruby koans [1] the other day.
I do have quite a bit of experience in programming, especially with functional language like Scala. I think it will be an interesting
journey to get familiar with Ruby. It is very different from what I did for the last 6-7 years. I'll try to share my impressions on language design, frameworks and anything that I find interesting.

### Ruby Ecosystem

First, even before starting this tiny project, I've noticed that Ruby open source ecosystem is massive.
The amount of Gems they have is nothing I've seen before. It certainly is bigger than Scala's. I think even if we 
add Java OSS, it still falls short when compared to Ruby.

That's pretty cool. In Scala, if you deviate from the mainstream even a little bit – you are stuck 
with debugging libraries you use, or you have to write your own. In Ruby, it seems like you can find a gem for anything.

I am not sure how hard is it to get help with some niche problems, but I guess I will find that at some point.

## Weather informer

### Initial concept

It was quite arid for a few weeks. Dust everywhere and no rain. I needed a tiny project to get started with Rails and this just fits the theme.
If you are not building something – you have a very different mental model from when you are building. It makes you learn faster, helps you focus on important bits and ignore the rest.

Initially I've come up with a simple concept:
- Download weather information from meteo.lt API
- Store it in a database
- Run some background job every X minutes
- Check if user needs to be notified about upcoming rain
- Notify through email

After I've built it, I noticed that it is not very useful and there are too many notifications. 
I've changed the requirements a bit and added a chatbot to the mix. I think its more useful and it became the main way I interact as well.

### With chatbot
Simple weather forecast informer. Will send a notification if weather conditions are met.

- Call meteo.lt API to get weather data
  - Save data to database
- Run periodic worker which checks if data matches specified conditions
  - Ex: if it will rain today – send a notification
    - Email
    - Telegram bot
- Telegram API to ask about current conditions or forecast
  - Will return a list of options to choose from.
    - Kada lis?, which returns nearest rain forecast
    - Atnaujink duomenis, which will update data
    ![Bot example](/images/2023/weather-informer/bot_example.png)
    - orai Will return warnings about today's weather
      ![Bot example](/images/2023/weather-informer/warnings.png)
      - translation: "rain today @ 9:00, rain @ 12:00, windy @ 12:00 with gusts at 10m/s"
- Select city
  - This defaults to Kaunas. No functionality implemented for switching

Note: this application is not intended for real world use. It is just a learning project.

## Architecture

### Overview of components

Entrypoints:
- Telegram chatbot
  - Telegram bot worker runs asynchronously and handles all messages incoming from Telegram API.
    - Note: it is not recommended to have long-running processes like this.
  - It will respond to messages and send notifications
  - Callback to refresh data will trigger worker and will download new data from meteo.lt. Runs synchronously and sends success message to the user
- Sidekiq schedulers
  - Runs every 24 hours and checks if any of the interesting conditions are met. If there are – notifies user through an email or telegram bot. Note: I've disabled this functionality, will update it to use Telegram later on.
  - Data downloader. There is no scheduler specified, but a good candidate to run data refresh job every few hours to keep data fresh

![Overview](/images/2023/weather-informer/overview_weather_informer.png)

### Notes on asynchronous processes

It seems canonical way to run asynchronous processes in Ruby is to run them in sidekiq and similar workers. It is a bit weird
for me coming from Scala. I would just run these processes on a different fiber and keep it in same process. No need to run these
side containers which connect to main application and keep state in yet another redis container! Process to process
communication would have to be done through redis if my process is asynchronous. I guess it is a tradeoff between simplicity and scalability.

### Database

Picked the simplest database possible – sqlite. In database there are two tables:
- places
  - Stores information about places of forecasts
- forecasts
  - Stores information about forecasts, basically 1:1 to what meteo.lt returns

```sql
create table forecasts
(
    id                          INTEGER     not null
        primary key autoincrement,
    place_id                    INTEGER     not null
        constraint fk_rails_716332ddb4
            references places,
    forecast_creation_timestamp datetime(6) not null,
    forecast_timestamp          datetime(6) not null,
    air_temperature             float,
    feels_like_temperature      float,
    wind_speed                  float,
    wind_gust                   float,
    wind_direction              float,
    cloud_cover                 float,
    sea_level_pressure          float,
    relative_humidity           float,
    total_precipitation         float,
    condition_code              varchar,
    created_at                  datetime(6) not null,
    updated_at                  datetime(6) not null
);

create index index_forecasts_on_place_id
    on forecasts (place_id);

```

There is also redis container. Redis is used by sidekiq to coordinate workers. I have also used it to store messages which need to be sent out by telegram bot.

### Data downloader

I've picked `HTTParty` gem. Super simple to use. Since Ruby is dynamic, I don't have to write any protocols for the expected data.
I guess I am supposed to pass role of validation further down. In my case, I am keeping same structure in DB, so I can re-use model validation and table constraints.

```ruby 
  # External API client, code=place
  def get_forecast(code)
    self.class.get("/#{code}/forecasts/long-term", @options)
  end
```

I am not sure how to handle errors canonically in Ruby yet. I think this should do for now.

```ruby
  # Service method
  def fetch_forecasts
    begin
      forecast = @meteo.get_forecast('kaunas')

      if forecast.success?
        response = JSON.parse(forecast.body)
        create_forecast(response)
      else
        Rails.logger.error "ForecastDownloadService error: #{forecast['error']['message']}"
      end

    rescue StandardError => e
      Rails.logger.error "ForecastDownloadService error: #{e.message}"
    end
  end
```

### Weather conditions monitoring

There is a service to monitor if certain weather conditions are met.
I think it would be interesting to see if there is anything special about upcoming day.

Numeral ranges to check if weather is special.
```ruby
  HOT_TEMP_RANGE = 25...30
  CLEAR_SKY_HOT_TEMP_RANGE = 24...29

  SCORCHING_TEMP_RANGE = 30..100
  CLEAR_SKY_SCORCHING_TEMP_RANGE = 29..100

  CLEAR_SKY_RANGE = 0..30
```
Check if any of the conditions are triggered. If they are – return them and they can be passed to the user.

```ruby
  def find_triggers(forecasts, conditions)
    conditions.flat_map do |condition|
      forecasts.reduce([]) do |triggers, forecast|
        case condition
        when :rain
          triggers << { :rain => forecast } if will_rain?(forecast)
        when :hot
          triggers << { :hot => forecast } if is_hot?(forecast)
        when :scorching
          triggers << { :scorching => forecast } if is_scorching?(forecast)
        when :windy
          triggers << { :windy => forecast } if forecast&.wind_gust >= 10
        else
          puts "Unknown condition: #{condition}"
        end
        triggers
      end
    end
  end
```

### Telegram bot

As mentioned, Telegram bot runs in a worker on sidekiq. I've read that it is not recommended to run long-running processes like this.
However, it works and I don't want to over-engineer this project any more than I need to.

It uses 'telegram-bot-ruby' gem. Works great.

```ruby
Telegram::Bot::Client.run(token) do |bot|
      Thread.new do
        bot.listen do |message|
          if message.from.id.to_s.in?(allowed_chat_ids)
            logger.info "Received message: #{message}"
          else
            logger.info "Received message from not allowed chat: #{message}, #{message.from.id}"
            next
          end

          case message
          when Telegram::Bot::Types::Message
            case message.text
            when /kaunas/i
              @bot_service.kaunas_selected(bot, message)
            when /orai/i
              @bot_service.weather_today_selected(bot, message)
            when '.'
              @bot_service.dot_selected(bot, message)
            end

          when Telegram::Bot::Types::CallbackQuery
            @bot_service.button_callback(bot, message)
          end
        end

        Thread.new do
          loop do
            message_info = RedisConnection.current.lpop('telegram_messages')
            if message_info
              send_notification(bot, message_info)
            else
              sleep 5
            end
          end
        end
      end
    end
```

At the bottom there is a loop which checks if there are any messages in redis queue. If there are – it will send them to the user.

Poor man's pub/sub.

These notifications are sent from sidekiq worker which is scheduled to check 'today's weather' every 24 hours and notify user afterward.
Emails suck, so I've added telegram notifications as well.

### Deployment

There is docker-compose.yml file which can be used to run the application. It will run 3 containers.
I will not get into detail, feel free to check it out in the repo itself.

## Conclusion

I've learned a lot about Ruby and Rails.
I was surprised how easy was it to pick up Ruby and Rails. I guess it is a testament to how good the community is.
Some things were very quick to code in: like 3rd party API client. ActiveRecord is also very nice to use to get something done quickly. Although I am used to writing queries myself. It will be interesting to do a more complicated project and see how it goes.

I didn't use Rails for its intended purpose: as HTTP server with templates and so on. I think I didn't experience all the benefits yet. However, I did get familiar with them and that achieved my own goal.


[Project repository in Github](https://github.com/benetis/weather_informer)

## References
- [1] [Ruby Koans](http://rubykoans.com/)








