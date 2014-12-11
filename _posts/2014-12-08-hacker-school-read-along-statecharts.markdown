---
layout: post
title: "Statecharts: A Visual Formalism For Complex Systems"
permalink: hacker-school-read-along-statecharts
category: hsreadalong
date: 2014-12-08 21:05
published: true
---

Last week's Hacker School Paper of the Week, [_Notation as a tool of thought_] \([Iverson]), stated that good notation has (at least) the following characteristics:

- Ease of expressing constructs arising in problems.
- Suggestivity.
- Ability to subordinate detail.
- Economy.
- Amenability to formal proofs.

This week's paper, [_Statecharts: A Visual Formalism For Complex Systems_] \([Harel]), details a notation for specifying complex systems.
By Iverson's characteristics (which he notes are not exhaustive, but certainly necessary), it is (almost[^1]) a _good_ notation.
Unlike the notation in Iverson's paper, Harel's notation is predominately _visual_[^2].
Here is a cropping of the Statechart that describes the author's digital wristwatch:

![Statechart example](/images/statechart-example.png)

The arrows whose tails are dots denote the entry point of that subchart.
In this case, the watch is initially dead; when the battery is inserted, the light is initially `off`; the power indicator is initially `OK`.

Arrows, on the other hand, represent transitions, with the label alongside the arrow serving to document the event causing the transition.
In this diagram, `b` indicates that the **b** button was pressed, so we know that pressing **b** transitions the `light` from `off` to `on`.
When the battery weakens, the power indicator begins to `blink`.

Dashed lines denote concurrent subcharts, so we know that when the battery weakens, the power indicator will start to blink no matter what else is happening on the watch.
Perhaps it will do so while the wearer is pressing **b**.
It will definitely do so while _something else_ is happening on the watch, even if that something is merely ticking the time away.

The solid outline at the tip of the `batt. inserted` event indicates a subchart boundary, so _everything_ inside the left-hand solid boundary depends on the battery having been inserted.
This means that, if we were for some reason interested only in how the presence or absence of a live battery affected the watch, we could remove all of the subcharts on the left-hand side and have a more succinct but no less informative or correct picture.
One can imagine a tool to manipulate state charts would allow its operator to collapse and expand subcharts to hide or show detail as needed.

And by no means is the above diagram an exhaustive example of a Statechart's expressiveness; there are affordances available to represent more nuanced behaviors, and Harel presents several ideas in the paper for future enhancements.
But with just a few boxes, arrows, and labels, we have a clear picture of system dependencies, concurrent operations, initial states, and possible actions for a realistic example.

---

To me, the most exciting aspect of Statecharts is their potential to be not just _a_ description of a system, but the _primary_ description of the system.
From the Conclusion:

> It is quite fair to say that most existing visual description methods in computer science are predominately intended as aids...Here we are suggesting that _visual formalism_ should be the name of the game; one uses Statecharts as the formal description itself, with each graphical construct given a precise meaning...Textual representations of these visual objects can be given, but _they_ are the aids...and not the other way around.

I agree with Harel that visual formalism should be the name of the game, but a cursory scan of the state chart implementations [on GitHub] seems to indicate that it has not yet become the name of the game.
Still, there's some exciting work on interfaces which permit direct manipulation by folks like [Bret Victor] and [Toby Schachman]---[Shadershop] being a recent example---and I hope that more programmers put their expertise towards these efforts in the future.
For folks interested in doing so, I think Harel's paper is a great place to start.

[_Notation as a tool of thought_]: http://www.eecg.toronto.edu/~jzhu/csc326/readings/iverson.pdf
[Iverson]: http://en.wikipedia.org/wiki/Kenneth_E._Iverson

[_Statecharts: A Visual Formalism For Complex Systems_]: http://www.inf.ed.ac.uk/teaching/courses/seoc/2004_2005/resources/statecharts.pdf
[Harel]: http://www.wisdom.weizmann.ac.il/~harel/

[on GitHub]: https://github.com/search?utf8=%E2%9C%93&q=statechart

[Bret Victor]: http://worrydream.com
[Toby Schachman]: http://tobyschachman.com
[Shadershop]: http://tobyschachman.com/Shadershop

[^1]: To my eyes, it's missing an obvious amenability to formal proofs. On gut feel alone, it feels like one _could_ use _some_ Statecharts in the process of proof. But the author doesn't come out and say that _any_ Statechart lends itself to proofs, so I won't either.
[^2]: Though as far as I can tell it is not particularly _spatial_. That is, it paints a picture, but the picture seems to be primarily one of boundaries and arrows; there don't appear to be any sort of "diagram physics" at play.
