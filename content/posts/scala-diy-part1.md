---
title: "Writing Scala DIY style (part1)"
date: 2023-04-08
# weight: 1
# aliases: ["/first"]
tags: ["scala", "diy"]
author: "benetis"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Scala DIY series"
canonicalURL: "https://benetis.me/posts/scala-diy-part1"
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
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

# Introduction

I've been contemplating a project in which I recreate commonly used libraries to gain a deeper understanding of their functionality. Many Scala projects rely on a variety of libraries that offer unique features such as effect systems, streams, and other essential components. It's crucial to understand the purpose of each building block in a modern Scala project.

While most of these libraries are well-documented and provide ample information about their design choices, I believe that the most effective way to grasp the author's intent is by re-implementing the library yourself. This approach requires a thorough understanding of the trade-offs made during the design process. By exploring alternative trade-offs, we can potentially enhance the library's functionality and adapt it to suit a variety of needs.

I've observed that some players enjoy revisiting older games and introducing self-imposed constraints to make the gameplay more engaging. For instance, in Runescape, players may opt for the Ironman mode, which prohibits item trading and requires them to figure out how to acquire everything independently. This significantly alters the gameplay experience. Similarly, in older games like Diablo 2, hardcore players face the challenge of not dying, which also substantially impacts the gameplay dynamics.

I believe we can apply this principle of adding constraints and challenges to enhance our enjoyment and learning experience when working with programming languages like Scala. By setting limitations or creating unique scenarios, we can deepen our understanding and have fun in the process.

# Scala 2 vs Scala 3

Initially, I considered working with Scala 2, as it's a language I'm quite familiar with. However, given the ineviteble migraiton to Scala 3, I might as well take the opportunity to explore and learn Scala 3.

# Constraints

In the build.sbt file, the only dependencies included should be for testing purposes, specifically using the MUnit library.
Initially, the focus will be on exploring Scala 3 features, as I am new to this version of the language. Later on, we can consider removing testing dependency as well, further streamlining the project.

# First non-direct goal

Although the primary objective is to re-implement libraries, it's essential to have real-world application scenarios to put these libraries into practice. One suitable application to consider as a starting point is a Memory Flashcards app.

## Questions
### In Scope
- Add flashcards
    - Question
    - Answer
- Review flashcards
    - Select how hard the answer was:
    - (again), (hard), (ok), (easy)
    - Depending on the answer, app will adjust metadata on card when card becomes "due for review" again
    - Review all cards which are due 
- Command line only. Control by capturing input in command line
- CRUD to a local file
### Out of Scope (for now)
- Multiple decks
- SQLite

### UI functionality
- Clear visible console
- Show card question
- Show answer after clicking space
- Show _buttons_ available for selection with "next due date" on them
- After clicking next -> clear and show next question

# Missing dependencies

Before diving into the flashcards application, there are a few dependencies that need to be addressed. While it's possible to create the application without them, the abstractions they offer can be beneficial.


- IO: Representing programs as 'data' simplifies the development process. Investing time in creating an IO library is worthwhile, as it will prove beneficial for various tasks in the future.
- CLI render library: Developing a more efficient CLI library is achievable and will allow rendering and working with command-line interface. The library can be crafted to align with a functional programming style, taking into account that CLI operations can produce side effects.
- File open/close -> safe resource acquisition and release: The IO library can incorporate these features simplifying the workflow.

For the initial version, we can follow a straightforward approach for data persistance:
    - Verify if the cards file is open (by checking if a .lock file exists).
    - Open the cards file.
    - Create a lock file.
    - Read all contents into memory.
    - Whenever a card or metadata is updated, flush the changes to the file.
    - Upon exiting the application, remove the .lock file.

_The custom dependencies developed for this project may not be as refined as some community libraries. However, that's not an issue, as the primary aim isn't to match or surpass the quality of existing libraries. The focus remains on learning and understanding through re-implementation. Later, as goals are updated, dependencies will be improved as well._

## Effects (or IO)

Firstly, regarding IO, there are two main aspects I want to focus on:
- Deferring computations by encapsulating them in IO. This abstraction is particularly valuable when writing code, as it enables better control over the execution flow.
- Managing and handling errors that may arise during the computations, allowing for more robust and resilient code.

Simple IO libraries have been covered quite a lot by Scala bloggers. Therefore, I think, there is no need to comment the DSL. DSL can be checked out at [Github](https://github.com/benetis/didactic-computing-machine/blob/master/software-and-math-exercises/scala3-diy/src/main/scala/io/IO.scala)

In future, this effect _library_ will get a lot of upgrades and we can go over them then.

## Resource acquisition and release

This can be modelled as a function on IO:

```scala
  def acquireAndRelease[A, B](
      acquire: IO[A],
      release: A => IO[B],
      use: A => IO[B]
  ): IO[B]
```

This way file closure can be guaranteed and is easy to use in functional paradigm as opposed to _finally_.

## CLI render library requirements from flashcards

It's uncertain how the final outcome will appear, but aiming for a one-directional approach. This can be structured as a currentState and a nextState. Here's a rough plan for what is needed:
- Ability to clear current console
- DSL for elements which will be displayed
    - Text(text, properties)
    - Div(elements, properties)
- Interpreter to render elements
- Rendering needs to be fast
- Colour support
- Capture keyboard inputs

### Questions
- How does render happen and how is output flushed to terminal?
    - Render interprer responsible for handling all elements and user input
- Performance limitations. Can this approach work at all?
- How is view handled from flashcards?
    - How generic should this CLI library be?
    - It would be best if all changing parts can be modelled as parameters
    - All elements would be functions which output something based on those paremeters
- Does terminal size needs to be known?
    - I took a stab at this, it seems its a bit complicated to get it working well. I think, we can get away without it
    
#### To be continued...



