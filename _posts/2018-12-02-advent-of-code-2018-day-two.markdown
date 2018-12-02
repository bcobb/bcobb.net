---
layout: post
title: "Advent of Code 2018, Day Two"
seo:
 title: "Advent of Code 2018, Day Two"
 description: "Talking through my solution to the second day of Advent of Code 2018"
published: true
slug: advent-of-code-2018-day-two
category: tbd
date: 2018-12-02 13:54
modified: 2018-12-02 13:54
---

Today: take a list of strings and find those which contain any letters that appear exactly two or three times.
Multiple the number of strings that have exactly two of any letter by the number of strings that have exactly three of any letter, and take that as the "checksum" of the list.

I decided to compute "histograms" of the given strings: maps from a character to the number of times it is repeated within the string.
This led me to learn that maps work in Reason much like [Sets do][yesterday] -- they're functors!

Building the histogram took a little head-scratching.
I knew that I wanted to add new characters into the map with a value of 1, but update the values for existing characters to be their current value, plus one.
I was a little concerned about using [`Map.Make.find`][find] because it throws an exception, but then I learned that exception handling in Reason is done with pattern matching, so it doesn't feel clunky

```reason
let characterHistogram = (chars: array(char)) =>
  Array.fold_left(
    (map, aChar) =>
      switch (CharMap.find(aChar, map)) {
      | item => CharMap.add(aChar, item + 1, map)
      | exception Not_found => CharMap.add(aChar, 1, map)
      },
    CharMap.empty,
    chars,
  );
```

With the histogram squared away, [solving the the rest of the problem][firstsol] required turning strings into histograms, and counting histograms that contained a value (or "binding" in the Reason nomenclature) of 2 or 3.

Only when I started on my Clojure solution did I realize that it's not necessary to keep any information about _which_ letters appear 2 or 3 times, so I could have gotten away with using a Set.
But I'm glad I used the histogram approach, because I got to learn about how Maps work.

The second problem asserts that there are only two strings in the corpus which differ by one letter in one position (e.g. `"brian"` vs `"bryan"`) and it asks for the letters that they share in common.

I ended up zig-zagging in [my solution][secondsol] for this one.

At a high level, I knew that I wanted to turn the list of words into a list of all combinations of _pairs_ of words, find the pair that differed only by one character, and return the letters they shared in common.

Calculating the difference, and finding the two words that had a difference of 1 was pretty straightforward.
But I struggled with turning that pair into a string of common letters.
Mainly, I kept balking at writing a function that felt like it was duplicating too much of my `positionalEqualities` function.
This led me to temporarily turn all of my `Array`s into `Lists` so that I could `List.filter`, but ultimately I reverted that change and decided to suffer the duplication to solve the problem.

If I was to go back and refactor, I would take a similar approach to my [Clojure solution][clj]: reify the concept of `pairwise` words and pass _those_ around instead of "combos" of the original words.

[find]: https://reasonml.github.io/api/Map.Make.html#VALfind
[yesterday]: /advent-of-code-2018-day-one/
[firstsol]: https://github.com/bcobb/advent-of-code-2018/blob/0c4d587b38cdd053f6c7e2d0995e8e4e6ad4c2f8/src/reason/Two.re
[secondsol]: https://github.com/bcobb/advent-of-code-2018/blob/9c83ba0cec34170a02cf17ebd51ad4e1105347a0/src/reason/Two.re
[clj]: https://github.com/bcobb/advent-of-code-2018/blob/8f2fa8e76ea655eef253d13a153872cf4629306b/src/clj/two.clj
