---
title: "Backtracking: 8 Queens"
date: 2020-04-05
# weight: 1
# aliases: ["/backtracking-8-queens"]
tags: ["computing", "scala"]
author: "benetis"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "An exploration into problem-solving with evolutionary computing, focusing on the 8 queens puzzle and backtracking algorithm."
canonicalURL: "https://benetis.me/posts/backtracking-8-queens"
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
    URL: "https://github.com/benetis/benetis.me/blob/master/content/posts/backtracking-8-queens.md"
    Text: "Suggest Changes" # edit text
    appendFilePath: false # to append file path to Edit link
---

# Introduction

Recently I started reading a book about evolutionary computing. As you can guess - its about problem solving using evolution concepts. One of the first examples in the book is about 8 queens puzzle. Its a puzzle in constraint satisfaction problem space and my guess is that it is solved with evolutionary computing later on. I think it would be interesting to look into more common algorithm for this puzzle before proceeding with the book. It is called backtracking.

# Puzzle

Put 8 queens on a 8x8 chessboard in such a way that no queen can take each other. It might sound an easy puzzle to solve with code. However if you give it a shot - you will quickly see that it is not so trivial.

![One of the solutions to this puzzle](/images/2020/backtracking/8queens.png)

# Approaching the problem

## Data structures

```scala
case class Square(x: Int, y: Int)

type PlacedQueens = Vector[Square]

val wholeBoard: Vector[Square] = {
  Range
    .inclusive(1, 8)
    .flatMap(x => {
      Range
        .inclusive(1, 8)
        .map(y => {
          Square(x, y)
        })
    })
    .toVector
}
```

Instead of usual chess squares with letters and digits, pair of digits (x, y) is used. This is to avoid mapping letters to digits for diagonal checks later on. This optimisation's problem's goal is to find a vector of square where queens can be put. Board of is chess is just a matrix with pairs of row and column numbers as their values.

```
| (1,1) (1,2) ... |
| (2,1) (2,2) ... |
```

## Algorithm

The first 'solution' that comes to mind is a recursive one. Try to put one queen and the next after that. No need to carry so called state of what squares were checked already and which ones were not. This results with a problem that if no solution is found with 8 queens and first queen on square (1,1).

Since this is not enough, next step is to keep some sort of state. The easiest solution should be to just iterate through all squares and start backtracking from each of the squares. This actually works, although the code below is not performant due non-early return of foldLeft (takes a minute~ to find a solution). Next step would be to replace it with something that can do early returns, but this is beyond my interest.

```scala
def placeQueens(toPlace: Int,
                available: Vector[Square]): Option[PlacedQueens] = {

  if (toPlace <= 0) {
    Some(Vector.empty[Square])
  } else {
    val result = available.foldLeft(Vector.empty[Square])(
      (prev, queenToPlace: Square) => {
        val afterQueen = canBePlaced(queenToPlace, available)

        afterQueen match {
          case Some(nextSquares) =>
            placeQueens(toPlace - 1, nextSquares) match {
              case Some(value) =>
                queenToPlace +: value
              case None => prev
            }
          case None => prev
        }

      }
    )

    if (result.isEmpty)
      None
    else
      Some(result)
  }
}
```

# Bibliography

- Introduction to Evolutionary Computing By A.E. Eiben, J.E. Smith
- [Demo'ed code of this problem (github)](https://github.com/benetis/didactic-computing-machine/blob/master/software-and-math-exercises/constraint-satisfaction/src/main/scala/me/benetis/Main.scala)
