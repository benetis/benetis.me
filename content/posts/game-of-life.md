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

## Rules of the game

Universe is a 2D infinite space. With that we have four simple rules for our automation:

1. Any live cell with fewer than two live neighbors dies, as if by underpopulation
2. Any live cell with two or three live neighbors lives on to the next generation
3. Any live cell with more than three live neighbors dies, as if by overpopulation
4. Any dead cell with exactly three live neighbors becomes a live cell, as if by reproduction

Each cell will have 8 neighbors which need to be checked for these rules to be used. This results in the next game state and the same rules will need to be applied to create the third game state. And so on.

```
+-----+
|YYY--|
|YXY--|
+YYY--+
```
where X is the cell and Y are its neighbors.

## Defining the task

In order to see how the simulation works two things are needed: 1) code to generate the next state 2) code to render state. The first part is where the rules of the simulation are implemented. For this my plan is to implement them using TDD methodology where I write the least amount of code possible. For the second part, the state just needs to be rendered. I think it's a good place to use HTML Canvas and just paint rectangles to represent cells.

## Test driven development

Starting with the simplest test and then implementing the code.

```scala
test("cell should die if it has no neighbours") {
      /*
      +---------+
      |---------|
      |---------|
      |-X-------|
      +---------+
       */
      val state = Set(Point(1, 1))

      assert(GameOfLifeRules.nextState(state))(equalTo(Set.empty[Point]))
}
```

```scala
test("dead cell with 3 live neighbours becomes a live cell") {
      /* Input
      +---------+
      |---------|
      |-Y-------|
      |Y-Y------|
      +---------+
       */
      /* Expected
      +---------+
      |---------|
      |-Y-------|
      |-X-------|
      +---------+
       */
      val state = Set(Point(0, 1), Point(1, 2), Point(2, 1))
      val expected = Set(Point(1, 2), Point(1, 1))

      assert(GameOfLifeRules.nextState(state))(equalTo(expected))
}
```

To keep this as short as possible - other tests can be found [in github](https://github.com/benetis/didactic-computing-machine/blob/master/software-and-math-exercises/game-of-life/src/test/scala/me/benetis/GameOfLifeRulesSpec.scala)

## Will it render?

Few things to be done to render cells. First is to decide on frameworks/tools and platform. 'Platform' was mentioned in the previous section - HTML Canvas. With HTML Canvas comes Javascript and ScalaJS is a perfect candidate there. I did consider Swing and JavaFX, but after playing with

it a bit I was not impressed. It seemed hard to do anything 'functionally'.

```scala
val program: ZIO[Clock, Throwable, Unit] = {
  for {
    renderer <- prepareScreen(config)
    refState <- Ref.make(RandomState.randomCenter(config))
    _ <- (render(renderer, config, refState) *> updateState(refState) *> UIO(
      dom.console.log("tick")
    )).repeat(Schedule.fixed(1.second)).forever
  } yield ()
}
```

```scala
private def render(renderer: CanvasRenderingContext2D,
                   config: RenderConfig,
                   ref: Ref[LifeState]): ZIO[Any, Nothing, Unit] = {
  for {
    _ <- clearCanvas(renderer, config)
    state <- ref.get
    _ = state.foreach(p => renderPoint(renderer, config, p))
  } yield ()
}

private def updateState(ref: Ref[LifeState]): ZIO[Any, Nothing, Unit] =
  for {
    _ <- ref.update(state => GameOfLifeRules.nextState(state))
  } yield ()
```

Everything is compiled to Javascript and rendered in a browser through index.html and game-of-life-fastopt.js

## Summary

This concludes this quick write up about my experiment with Conway's game of life implementation. Two conclusions I made: Test driven development can really improve code to prevent over engineering solutions and simulations are very interesting.

Missing code code can be found [in github](https://github.com/benetis/didactic-computing-machine/tree/master/software-and-math-exercises/game-of-life)

### Bibliography

- [0] - [https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life)
- [1] - Type-safe, composable asynchronous and concurrent programming for Scala
- [2] - [Github link to the sources](https://github.com/benetis/didactic-computing-machine/tree/master/software-and-math-exercises/game-of-life)

### Feedback

If you have any suggestions or want to contact me - I am eagerly waiting for your feedback.
