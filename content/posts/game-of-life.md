---
title: "Conway's Game of Life"
date: 2020-05-04
# weight: 1
# aliases: ["/game-of-life"]
tags: ["computing", "scala", "zio", "simulation"]
author: "benetis"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "This post explores implementing Conway's Game of Life using Scala and ZIO library with a Test-Driven Development (TDD) approach."
canonicalURL: "https://benetis.me/posts/game-of-life"
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
    URL: "https://github.com/benetis/benetis.me/blob/master/content/posts/game-of-life.md"
    Text: "Suggest Changes" # edit text
    appendFilePath: false # to append file path to Edit link
---

# Introduction

The game of life is a cellular automation devised by John Conway. This simulation has only a few rules, but it is Turing complete [0] and incredibly interesting to watch. With recent news about Conway, I thought it's time to give it a go. In one programming kata, I had encountered this problem, but never went to fully complete this simulation. This time, I'll use my favorite programming language, Scala, and my current favorite library, ZIO [1].

![Game of Life Simulation](/images/2020/game-of-life/animation.gif)

