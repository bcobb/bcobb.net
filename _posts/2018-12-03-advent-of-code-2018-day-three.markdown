---
layout: post
title: "Advent of Code 2018, Day Three"
seo:
  title: "Advent of Code 2018, Day Three"
  description: "Advent of Code 2018, Day Three"
published: true
slug: advent-of-code-2018-day-three
category: tbd
date: 2018-12-03 21:06
modified: 2018-12-03 21:06
---

The challenge today was based around rectangular "claims" which are strings formatted like:

```
#123 @ 4,5: 6x7
 ^^^   ^ ^  ^ ^
 |     | |  | |
 |     | |  | |
 |     | |  | +-- height of rectangle
 |     | |  +-- width of rectangle
 |     | +-- y coordinate of top-left of rectangle
 |     +-- x coordinate of top-left of rectangle
 +-- claim id
```

The rectangles described by the claims live on a grid of 1x1 squares.

Given a bunch of these claims, we're asked to:

1. Find the number of squares which are members of two or more claims
2. Find the ID of the claim which is comprised of no shared squares

I wasn't able to start on the first problem before I had to leave for work this morning, but on my walk to the train I realized that I could turn each claim into a set of coordinates, each coordinate representing one 1x1 square, and then turn the coordinates into a histogram.
For the first problem, I'd just need to count the bindings in the histogram with a value of 2 or greater.

I was glad I spent some time with `Map` yesterday; today I got to dig one level deeper and use `Map.Make` with a module of my own (`Point`, with a type `t` that corresponded to the `(x,y)` coordinate pair).
It wasn't immediately obvious that I could use `Pervasives.compare` as my map's `compare` function, but I convinced myself it would work by futzing around with `Js.log`.

With the `claims -> points -> histogram` pipeline in place, getting the first solution was a `filter` and a `cardinal` (i.e. the `Map` version of `length` or `count`) away.

The second solution used a lot of the same primitives from the first solution, which made me feel good about the first solution!

Where in the first solution we needed to find bindings in the histogram with values of 2 or greater, we now need to find those with a value of 1.
These are our "solitary" points.
To find the answer, we need to find the claim which is comprised entirely of solitary points:

```
claims
|> List.find(claim =>
     points_of_claim(claim)
     |> List.fold_left(
          (every, point) => every && PointMap.mem(point, solitary_points),
          true,
        )
   );
```


After today, there are a couple things I need to look into:

1. is there an idiomatic/safe way to add functions to a module within a certain scope?
I would have liked to implement `List.every` to declutter the body of the function for the second solution.
1. what is the story around lazy collections in Reason/OCaml?
One thing I like about [my Clojure solution][clojuresolution] is that it's not necessary to distinguish between `find` and `filter | first` thanks to lazy collections.

And I need to experiment more with the pipe operator to get a better feel for when it helps or harms readability.

[reasonsolution]: https://github.com/bcobb/advent-of-code-2018/blob/f1784a85d6e9fbd3656698959a06677b08d2f290/src/reason/Three.re
[yesterday]: /advent-of-code-2018-day-two
[clojuresolution]: https://github.com/bcobb/advent-of-code-2018/blob/f1784a85d6e9fbd3656698959a06677b08d2f290/src/clj/three.clj
