---
layout: post
title: "The Power of Interoperability: Why Objects Are Inevitable"
seo:
  title: "The Power of Interoperability: Why Objects Are Inevitable"
slug: hacker-school-read-along-the-power-of-interoperability
category: hsreadalong
date: 2014-10-20 09:30
published: true
---

This week's [Hacker School Paper of the Week](https://www.hackerschool.com/blog/48-paper-of-the-week-the-power-of-interoperability-why-objects-are-inevitable) is Jonathan Aldrich's [_The Power of Interoperability: Why Objects Are Inevitable_](https://www.cs.cmu.edu/~aldrich/papers/objects-essay.pdf).
I remember seeing it linked on Twitter some time ago, when someone excerpted this:

> I have too much respect for the many brilliant developers I have met in industry to believe they have been hoodwinked, for decades now, by a fad.

A fair sentiment, I think.

Aldrich doesn't argue that objects are a better or worse tool for data abstraction than ADTs (as David says in the introductory post, this paper really does complement [last week's](/hacker-school-read-along-on-understanding-data-abstraction/)); rather, he argues that objects emerge naturally out of a need to create "service abstractions."
For instance, objects lend themselves well to plugin architectures: to "plug in" a new capability, one need only supply an object that responds reasonably to a set of messages sent by the system to which it is plugged.
Crucially, the plugged-in object could very well be a whole service in its own right, so long as it responds to the right messages in a reasonable way---hence, service abstraction.
_This_ is the interoperability---the interoperability between systems, and not just between data structures---that makes objects so powerful.
(My favorite analogy for interoperability between systems is "gears that mesh." What a great visual.)

Frameworks often lean on the interoperability afforded by objects to abstract over application architectures.
Rails, for instance, uses the first-class interoperability of objects to abstract over database connection pooling services for a variety of engines.
By virtue of this, it is possible (though, in my experience, not trivial!) for a consumer of a framework to write such a service of their own.

As I was reading the section on frameworks, the sentence "[F]rameworks typically must support _unanticipated_ extension" made me sit back and reflect on framework design.
When is a framework _done_?
(This is maybe more of a question for benevolent dictators than it is a general question of design.)

A lot of things clicked for me in the conclusion of the paper:

> [The interoperability afforded by objects] cannot be duplicated in other programming paradigms without likewise creating service abstractions, thus simulating the essence of objects.
> ...
> [A]n argument for object-orientation need not be an argument against other paradigms. Good support for service abstractions is compatible with good support for ADTs, for functional programming, and for many other desirable language features.

Following that last sentence especially, I would be remiss to not mention Gary Bernhardt's talk ["Boundaries"](https://www.destroyallsoftware.com/talks/boundaries) again.
I'll also add that [Brian Marick](http://exampler.com/)'s book, [_Functional Programming for the Object-Oriented Programmmer_](https://leanpub.com/fp-oo) shows a rudimentary object system implemented in Clojure.
I highly recommend checking it out.

Aldrich closes with some predictions based on his ideas, as well as with some thoughts on potential future work. In the same spirit, here are some ideas that, having read this paper, I need to explore further:

1. The idea that OO might provide _psychological_ advantages (and that these are somehow distinct from _technical_ advantages).
1. The idea of a rigorous design language (I want to make rigorous arguments about what's possible and what's not possible in a given paradigm---I'm not totally convinced by the arguments I've read thus far in these sorts of papers.)
1. Newspeak-style modules. These are mentioned a couple of times in the paper, but I'm not familiar with them.
