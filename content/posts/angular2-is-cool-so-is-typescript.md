
---
date : "2016-11-24"
draft : false
title : "Angular 2 is cool. So is Typescript"
slug : "angular2-is-cool-so-is-typescript"
tags : ['Angular2', 'Frontend']
banner : ""
aliases : ['/angular2-is-cool-so-is-typescript/']
menu:
    main:
        parent: 'angular'
---

##### Introduction

Angular framework for building applications. Let's do a review over its important parts.

###### Language of choice - Typescript

Angular was written in Typescript. As for Dart - Typescript was easily transpiled to Dart and that is how Dart version was maintained. It's just that Typescript code required some things for Dart version to work. At some point (this summer) - Angular team decided to split Angular Dart project to a separate one for better or worse. Motivation was to have better typescript version (performance, unnecessary code..) and more idiomatic Dart version.

Now for development - its a different story. You can use use:

* Pure Javascript (ES5)
* Typescript
- Dart
- ES2015* (lack of docs -> horror, no AOT support)

All there choices have separate Angular documentations - but as you can expect - Typescript's version is the best. If you are considering the language - _just go with Typescript_ unless you have some really specific needs.

##### Typescript - what is so great about it?

_It is a superset of ES2015 javascript_. Which means it will adapt to its specs and evolve to match javascript. It will not be a completely separate language but rather an extension of it. And since we are still running javascript in the browser (and getting runtime errors) - this is nice.

_It has optional types_ - this makes migrating from JS to Typescript easy - non-existing - raw JS just works. However, this also enables you to skip writing types whenever you want - which means on lazy day you will just say "meh, I won't need them.

_Refactoring and IntelliSense_ - Oh yes. This is one of the best things. You decide to change something for let's say naming consistency? No problem IDE will do job for you. Most of the thing IDE misses will still be reported by the compiler. Big codebase of Typescript - nothing to fear! - it can be refactored.

_Type thinking_ - You can just about types when coding - makes everything so much easier!
> f(x) transforms my x to f(x) - while it sounds to simple - this new ability almost instantly made all code produced by me with less side effect

taken from http://benetis.me/quick-glimpse-at-world-of-elm/


_Language features_ - classes, interfaces, type switching, strict null checks, encapsulation(sort of)... And so many more are coming!

_Raw Javascript just works_ - As mentioned above - you can compile any JS using the compiler and it will just work.

_Community is big_ - you won't be solving problems alone. Typescript is popular and I am confident you won't have problems finding answers.

*Why typescript? TLDR;* - static typing is good

##### Angular application architecture overview

What kind of file structure, components tree, libraries and etc. you find in Angular project?

First of all - The framework consists of several libraries, some of them core and some optional [0].

In this diagram you can see main building blocks of Angular application

![Angular2 architecture](/images/2017/02/overview2.png)

For best explaining - refer to Angular documentation -
https://angular.io/docs/ts/latest/guide/architecture.html

TLDR; below

**Components**

Your UI components. Contain the needed UI logic, styles are encapsulated. You input the data and get events from them. This makes understanding and testing them easier.
They are main building blocks of your application.

Code example #1

This is how our slide component currently looks - has two inputs.

slide.component.html
```typescript
<div class="slide">
  <div class="slide-title">{{title}}</div>
  <div class="slide-content">{{content}}</div>
</div>
```
This is how it is consumed in slides component. We pass both title and content as inputs of the component. We also have a button on which clicked next slide appears. We won't go into detail - if you want to - you can view it here. (Also tests exist) [github](https://github.com/benetis/enview/tree/master/src/app/slides)

What is important to note - you just pass data to slide component and that is the only way to communicate to that component. Another would be to receive an event from it. Components are separate units, which do their specific things. This is cool.

slides.component.html
```typescript
<ev-slide
  [title]="slides[currentSlide].title"
  [content]="slides[currentSlide].content"
></ev-slide>
<button class="next-slide" (click)="nextSlide()">Next slide</button>
```

**Modules**

Group your components, directives, services into groups so called NgModules. Component can only belong to one module. And module can import/export components (sort of encapsulation). Good way to split your application code. Router makes it easy to lazy load modules whenever you want them.

app.module.ts
```typescript
@NgModule({
  declarations: [ //This is where our components are assigned to this module
    AppComponent,
    SlidesComponent,
    SlideComponent
  ],
  imports: [ //Other modules our module is going to use. Must be here if you want to use anything from them
    BrowserModule,
    FormsModule,
    HttpModule
  ],
  exports: [
    SlidesComponent,
    SlideComponent
  ], //Component which can be imported into other modules
  providers: [SlidesService], //Providers to inject
  bootstrap: [AppComponent]
})
export class AppModule { }
```

**Services**

Services is the place where you your logic. It usually is a class with well defined purpose. (Tax calculator, logging, data fetching). Services can be injected into any component.


**File structure**

![File structure](/images/2017/02/Screen_Shot_2016_11_27_at_11_38_14.png)

As you can see all component related files are sitting together. Styles, tests, logic. It is easy to find things you need and you won't abandon your tests if they are so close to your code.

##### Handling state

To handle the annoying problem of cache invalidation at this moment you seem to only have two ways. One is to adapt some version of flux (redux version ngrx/store is popular in Angular community) and another is to use RxJs features like behavior subject.

As an example - in component you subscribe to `BehaviorSubject` and in state service you pass observable stream to that subject. What this achieves us - current value is stored in subject - letting us do easy caching.

##### Router

Let me begin with saying - I like working with Angular router. It is declarative, has cool features like lazy loading, is flexible - yet simple to understand.

Lazy loading - why it is relevant. *You just don't want to ship your users code they won't be using*. And as application gets bigger - this becomes more and more important - and Angular router through modules lets to that easily.

##### Dependency injection (DI)

DI is used everywhere. Providers are constantly injected into components. Angular makes it easy to do DI - because it is done through types (Typescript annotations). Also -  makes testing simple.

An example how easily you can stub a service in a test is just below.

![Testing DI](/images/2017/02/Screen_Shot_2016_11_30_at_17_12_46.png)

##### Mobile and offline friendly

To "convert" our application into mobile app - you can use NativeScript which is basically the same thing as React native is to react. This enables us to share application code between platforms.

Ionic 2 - we can just build everything using HTML/JS/CSS and it will work on mobile. Layouts will not be using native components.

Angular also brags that it will support touch events/ gestures of different devices

##### AoT compilation

One of the best features of Angular is that it has a compiler which will build our code once and we won't need the compiler again for it to work.

This leaves us with few cool things:

* App is built once
* Faster rendering - no compiler time
* Smaller application size (no need to drag that "huge" compiler together)
* Template errors are caught at compile time
* Better security
* Tree shaking (dependency graph walking)

##### Angular Universal (Server side rendering)

Advertised as it just works with a few gotchas (not interacting with DOM directly)

At the moment - https://github.com/angular/universal you can find .NET core and NodeJs examples.

Angular team said there will be improvements during next six months (two months ago)

##### Angular-cli

Command line interface for Angular applications. Let's you setup quick boilerplate, compile apps, run tests. Even generate components and etc. It is based on ember-cli. Main idea is to solve problem of complex configurations.

https://twitter.com/iamdevloper/status/787969734918668289

At this moment - it is not flexible and is slow. Promises to make it faster at near future.

##### Community

Community is great. Although not that big that moment - but we already have some great tools (Augury, ngrx stuff..)

There are also enough Stack overflow answer to live and develop. (Almost)

Lots of info and talks during conferences about it.

Seems like a case of Hype driven development, but I really doubt it will be abandoned - probably the opposite.
[](https://blog.daftcode.pl/hype-driven-development-3469fc2e9b22#.t3suy6t3h)

##### Angular 3.0

Angular versioning is stupid. They know it, we know it. As an example RC5 release introduced NgModules which is one of main building parts of application. Before that components had dependencies - you can imagine how much rewrite it introduced. Probably they were rushing to release 2.0 final ASAP since it has been developing for more than a year and rushed to RC too early. After that they got a lot of rant from developers who already had decent size applications written. With final release they adopted semantic versioning. Hoooraaaay! Anyway - changes to api (breaking) will only be introduced with major releases which should be every six months or so.

##### Bad parts

* Early stage - community is just learning - SO has answers, but not about best practices and etc.
* Bugs - they are decreasing rapidly but in more complicated components you are sure to find some comments saying this is workaround due to some issue
* Setting up is complicated (cli) - while angular-cli seems nice - for more complex applications you will want your own configuration. Oh and setting up project with Webpack and other cursewords is a nightmare
* Not Javascript oriented - has a lot of custom DSL (domain specific language)

###### Feedback

If you have any suggestions - I am eagerly waiting for feedback. [https://benetis.me/posts/contact-me/](/posts/contact-me/)

###### Sources

[0] - https://angular.io/docs/ts/latest/guide/architecture.html
