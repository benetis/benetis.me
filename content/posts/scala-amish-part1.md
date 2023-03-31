---
title: "Writing Scala in Amish style"
date: 2023-03-31
# weight: 1
# aliases: ["/first"]
tags: ["scala", "amish"]
author: "Me"
showToc: false
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "Scala Amish series"
canonicalURL: "https://benetis.me/posts/scala-amish-part1"
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

In the build.sbt file, the only dependency included should be for testing purposes, specifically using the MUnit library. 
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
    - Depending on the answer, app will adjust metadata on card when to be reminded again
    - Review all cards which are due 
- Command line only. Control by capturing input in command line
- CRUD to a local file, possibly SQLite
### Out of Scope
- Multiple decks

