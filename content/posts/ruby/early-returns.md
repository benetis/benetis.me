---
title: "Ruby early returns can be harmful"
date: 2023-08-23
tags: ["ruby", "pattern-matching"]
author: "benetis"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Ruby early returns can be harmful"
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
    URL: "https://github.com/benetis/benetis.me/blob/master/content/posts/ruby/early-returns.md"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

# Introduction

I've been using Ruby for a some time now and I've noticed that early returns can be over used.
I'll review at one example with enums where I think some early returns can be harmful.

# Enums

Ruby does not natively support enums, but they still do exist in business and other domains.
There are alternative ways to define them, let's look at the simplest one:

```ruby
module UserRole
  ADMIN = :admin
  USER = :user

end
```

Simple module with two constants. This is a very simple enum, but it can be used in a lot of places.
In a business domain it also shows us possible values for a user role.

# Working with enums

### Pattern matching

Let's write a simple function which greets the user based on his role:

```ruby
def greet(role)
  case role
  in UserRole::ADMIN
    puts "Hello, Admin!"
  in UserRole::USER
    puts "Hello, User!"
  end
end
# greet(UserRole::ADMIN) => Hello, Admin!
```

Quite concise, powerful and easy to read. 
But what if we want to add a new role? Let's say `MODERATOR`.

```ruby
module UserRole
  ADMIN = :admin
  USER = :user
  MODERATOR = :moderator
end
```

Now let's try greeting a moderator:

```ruby
greet(UserRole::MODERATOR) # => NoMatchingPatternError
```

Wonderful, we get an error. This is a good thing, because we know that we need to update our function.

### Old switch statement

What if we tried the same, just with `case` and `when` instead of `in`?

```ruby
def greet(role)
  case role
  when UserRole::ADMIN
    puts "Hello, Admin!"
  when UserRole::USER
    puts "Hello, User!"
  end
end

greet(UserRole::MODERATOR) # will fail silently
```

This is not good, because we don't get any errors and we don't know that we need to update our function.

However, we can use `else` to get an error:

```ruby
def greet(role)
  case role
  when UserRole::ADMIN
    puts "Hello, Admin!"
  when UserRole::USER
    puts "Hello, User!"
  else
    raise "Unknown role"
  end
end
```

This is better, but we do need discipline to use `else` every time. On top of that
we need specifically create an error, which is not very convenient. 
NoMatchinPatternError before was quite descriptive itself. 

### Early returns

Let's try to use early returns:

```ruby
def greet(role)
  return puts "Hello, Admin!" if role == UserRole::ADMIN
  puts "Hello, User!" if role == UserRole::USER
end

greet(UserRole::MODERATOR) # will fail silently
```

It is the problem – this will fail silently. Also, there is no `else` to save us.
We don't get any errors and we don't know that we need to update our function. As bad as it gets.

# Conclusion

Early returns can be harmful, because they can hide errors and make code more prone to bugs.
They can also help make functions simpler, but its not always the case.

I think that pattern matching [1] is the best way to work with data structures similar to enums.

# References
[1] - https://docs.ruby-lang.org/en/3.0/syntax/pattern_matching_rdoc.html



