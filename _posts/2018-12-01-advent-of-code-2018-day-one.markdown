---
layout: post
title: "Advent of Code 2018, Day One"
seo:
  title: "Advent of Code 2018, Day One"
  description: "Talking through my solution to the first day of Advent of Code 2018"
published: true
slug: advent-of-code-2018-day-one
category: tbd
date: 2018-12-01 14:05
modified: 2018-12-01 14:05
---

[Advent of Code][aoc] is a series of daily programming puzzles created by [Eric Wastl][wastl] and published daily in the month of December.
I [wrote one post on last year's challenges][2017], but this year would like to write about them more frequently, while the solutions are top-of-mind.
Do not read this if you would like to solve these challenges without any assistance!

For 2018, I plan to do all of the challenges in [ReasonML][reason].
I write a lot of JavaScript at my current job, and I thought it'd be interesting to see how it compares to my experience with Flow and TypeScript.
I hope to produce some Clojure solutions as well, as a point of comparison (and because doing it Clojure was a lot of fun last year).

The first problem required reading inputs representing "frequency changes" that look like `"-1"` and `"+43"`.
The goal is to determine the result of starting from a frequency of `0` and applying those operations.
For example, applying the two changes above would produce `42`.

Here is my [Reason solution][reasonsolution].

I decided early that I wanted to have a type to represent a "frequency change" and wrote a function that could take a `string` of one such change, and return a `frequencyChange` (something that I didn't realize prior to today was that ReasonML types _must_ start with a lower-case letter).

A `frequencyChange` is a two-ple of an `operation` and an `amount`. `operation` is an alias of `char`, and `amount` is an alias of `Int32.t` (as I understand it: `Int32` is a module, and `t` is the member of the module which returns the module's type).
My initial solution used `int` instead of `Int32.t`, but I switched to `Int32` while implementing the solution to the second problem, as I'll detail below.

The solution turns the raw input into a list of change strings, turns the change strings into `frequencyChange` two-ples, and then folds over the `frequencyChange`s to product the final frequency.
It might be clearer to just see the highest-level code:

```
inputLines()
  |> List.map(lineToFrequencyChange)
  |> List.fold_left(applyFrequencyChange, Int32.zero);
```

The second problem was a bit trickier: figure out the first frequency result to be computed twice, applying the list of changes as many times as is required to find such a repeat.
Clojure has a handy `cycle` function that would take the list of changes and return an infinite, lazy sequence of those changes, repeating _ad infinitum_ -- exactly the kind of sequence we'd like to iterate over to solve this problem.
Unfortunately, I did not see anything like that in the Reason standard library.

After some false starts, I ended up on the [API docs for Streams][stream] and figured out how to make a `Stream` that would behave like `cycle`, yielding the first item in a list after it yielded the last item in the list.

```
let circularStream = (l: list('a)): Stream.t('a) => {
  let length = List.length(l);

  Stream.from(i => Some(List.nth(l, i mod length)));
};
```

To keep track of which frequencies had been computed, I wanted to use a set.
I found the [Set documentation](https://reasonml.github.io/api/Set.html) a little confusing at first, but the example helped make things click: there's not a general-purpose set constructor or type.
Instead, there's a `Set.Make` "functor" which takes a type and returns a module that contains functions for operating on sets of that type.

The way I'm thinking about it is: rather than having a Set library, there's a Set library-maker, for any type that knows how to `compare` two of its values.

This led me to rewrite all of the `int`-based frequency expressions to use `Int32`; as far as I can tell, there's no way to use `Set.Make` with `int`.

With my cycling stream and a set module, I was able to write a `while` loop that takes items from the stream, computes the next frequency, and adds it to a set of known frequencies.
If the next frequency is already known, it's our solution.

---

Overall, I'm pretty sure there's a lot that's not quite idiomatic about these solutions, but I am happy to have found the correct results!

The two specific questions I have are:

1. is there a more direct way to turn the change strings into a frequency change that can be applied directly to a frequency?
   Follow-up: Is this what applicatives are for?
1. is there a more functional way to churn through the infinite Stream, collecting intermediate frequencies, and stopping when I find a duplicate?
   Related: is it possible to get away without using any `ref`s?

[aoc]: https://adventofcode.com/2018
[wastl]: http://was.tl/
[2017]: {{site.url}}/advent-of-code-2017
[reason]: https://reasonml.github.io
[reasonsolution]: https://github.com/bcobb/advent-of-code-2018/blob/201554580b0eaf9e14c5c979107c20c94e6f22a3/src/reason/One.re
[stream]: https://reasonml.github.io/api/Stream.html
