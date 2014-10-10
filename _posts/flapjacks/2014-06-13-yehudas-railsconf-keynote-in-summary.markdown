---
layout: post
title: Yehuda's Railsconf Keynote, in Summary
permalink: yehudas-railsconf-keynote-in-summary
published: true
date: 2014-06-14 17:23
category: flapjacks
author: Brian Cobb
---

The other day in DevChat, Tristan linked to [Yehuda Katz's keynote][1] from this year's RailsConf. I'd like to chat about the talk at some point, either in the Lounge or in the comments (or *gasp*, both!) so here is a (hopefully unbiased) summary (I sometimes use quotes for sarcasm, but here they are direct or near-direct quotes):

A number of years ago (2008, I think) at his own RailsConf keynote, DHH noted that "people like choices a lot more than they like having to choose." Yehuda refers to a body of research which seems to indicate that not only do people not like having to choose, but that making decisions results in "ego depletion," which in turn leads to worse choices (see: buying junk food while waiting in a grocery store line).

We can minimize ego depletion with psychological hacks, such as by identifying default choices. Frameworks such as Rails set defaults so that developers need not make so many choices that are not directly related to the business need they're trying to meet. Something else DHH said in an earlier keynote is that one thing Rails got right was "confessing commonality" -- the defaults that Rails provides are actually good enough for a wide variety of applications. Every application is not a snowflake; those that aren't don't need "unique and special tools."

Yehuda invokes Steve Jobs talking about managing software complexity, and highlights a construction metaphor Jobs used: scaffolding will, as it is built higher, eventually collapse on itself. Abstractions allow us to build scaffolding not from the ground, but from the 23rd floor.

We already build on a foundation of abstractions and frameworks. The modern development stack builds on decades of work that resulted in x86, C, POSIX, and so forth. As such, Yehuda is surprised at how strong the sense of snowflake syndrome is in the JS community. That because abstractions leaked once or leak a little bit means that they will always leak or that they should never be used. He quotes Jeff Atwood commenting that leakiness is inevitable, so an abstraction should be judged solely by its affect on the code (readability, maintainability, and so forth).

Every community starts from some foundation and goes through an area of experimentation where there are no frameworks, and there are no agreed-upon abstractions. Communities eventually need to escape the area of experimentation and agree on the good defaults and shared solutions that they can use as another layer in their foundation. This is why Rails has lasted 10 years and is a strength of the Rails community.

 [1]: https://www.youtube.com/watch?v=9naDS3r4MbY
