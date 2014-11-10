---
layout: post
title: "Managing Update Conflicts in Bayou"
permalink: hacker-school-read-along-managing-update-conflicts-in-bayou
category: hsreadalong
date: 2014-11-09 22:30
published: true
---

This week's Hacker School Paper of the Week is [_Managing Update Conflicts in Bayou, a Weakly Connected Replicated Storage System_](https://www.hackerschool.com/blog/53-paper-of-the-week-managing-update-conflicts-in-bayou-a-weakly-connected-replicated-storage-system).
From the paper, "Bayou is a replicated, weakly consistent storage system designed for...less than ideal network connectivity."
Two humble logs---the Write Log and the Undo Log---allow Bayou to handle poor network connectivity[^1].
As you might expect, each entry in the Write Log (a "Write operation") corresponds to a Write issued to the server.
Each Write operation _also_ knows two important things:

1. How to determine if it will conflict with an existing Write operation. In Bayou, this is called a _dependency check_.
2. An algorithm to construct an alternative, non-conflicting Write operation[^2] in the event there is a conflict. In Bayou, this is called a _merge procedure_.

The motivation for storing these two procedures is that it is difficult, if not impossible, for a Bayou server to behave intelligently with respect to an arbitrary domain.
So, instead of tailoring Bayou servers for use with a limited set of domains, Bayou servers can run arbitrary, domain-specific code supplied by their clients, which are embedded in the domain.

This sort of inversion of control is, to me, the big idea behind Bayou---it's what makes Bayou a general-purpose storage system.
Accordingly, I want to know more about how it works!

The authors mention that the merge procedures are written in Tcl, and then executed with a Bayou-specific Tcl interpreter, but they never discussed the rationale for using Tcl.
What made Tcl a good fit for the task?
What does it look like to modify a Tcl interpreter to impose the sorts of constraints outlined in the paper?
How fun would it be to write a Bayou server/client stub in Clojure or Elixir and just send quoted code around for both dependency checks and merge procedures?

---

This paper was fun to read, approachable, and a fertile bed for future project ideas---thanks to Maggie Zhou for suggesting it!
For those in the Bay Area, [Peter Bailis](https://twitter.com/pbailis) is giving a talk on the paper at the [December Papers We Love SF meetup](http://www.meetup.com/papers-we-love-too/events/197678922/).
Definitely check it out if you can!

[^1]: With better planning, a rough implementation of the Write Log-Undo Log-Tuple Store trifecta would have been a solid code contribution for the week.
[^2]: A simple non-conflicting operation might just be to insert an entry into some error table noting the problem. In practice, Bayou clients would probably rather provide a procedure that actually tries to resolve the conflict.
