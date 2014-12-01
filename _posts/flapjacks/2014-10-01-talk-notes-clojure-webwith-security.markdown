---
layout: post
title: "Talk Notes: clojure.web/with-security"
permalink: talk-notes-clojure-web-with-security
published: false
date: 2014-10-01 20:32
category: flapjacks
author: Brian Cobb
---

I've been keen on Clojure for some time and tend to Instapaper every mildly interesting post or talk related to the language and its ecosystem. One such talk that I used some bench time to watch was [this one][1], on the state of web application security in the Clojure ecosystem.

The TL;DW is that the security story is surprisingly weak, and Aaron, the speaker, gives some opinions as to how to rectify that (one is as simple as "use HTTPS, dummy!"). Based on a [corresponding HN comment][2], I'm not sure how much stock I'd put in *every* recommendation, but the overarching advice writ large is pretty good: make security easy by building simple, cohesive abstractions, with sensible (but configurable!) defaults where possible. **ETA**: He also notes that it is unwise to *assume* that because someone else wrote something that it is secure and/or complete and/or what you actually need. In particular, build abstractions to handle the following concerns:

1.  Password management
2.  Session management
3.  Authentication
4.  Authorization
5.  CSRF
6.  XSS
7.  SQL Injection
8.  Encryption
9.  Secure Headers

It sounds like Clojure falls short in Session Management, XSS, and Secure Headers in particular. Aaron mentions SQL Injection, too, but it sounded like he was more disappointed in the relative lack of progress on korma than he was concerned about it being easy to open yourself up to SQL Injection attacks. The reasons for falling short vary, and if you're curious about them I definitely recommend watching the talk. Even if his exact recommendations aren't sufficient, the problems he presents are quite real.

Given that an evergreen Growth Day pursuit is to explore new ecosystems, the above nine items seem like solid things to keep in mind when doing so, be it Go, Node, or even Ruby-off-the-Rails. This talk also spurred me to glance through the [OWASP][3] website, which I'd like to spend some more time digesting/discussing with anyone else who's interested.

 [1]: https://www.youtube.com/watch?v=CBL59w7fXw4
 [2]: https://news.ycombinator.com/item?id=7474989
 [3]: https://www.owasp.org/index.php/About_OWASP
