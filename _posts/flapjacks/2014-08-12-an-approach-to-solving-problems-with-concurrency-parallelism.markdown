---
layout: post
title: An approach to solving problems with concurrency/parallelism
seo:
  title: An approach to solving problems with concurrency/parallelism
  description: Thinking about concurrency/parallelism
slug: an-approach-to-solving-problems-with-concurrency-parallelism
published: false
date: 2014-08-12 13:01
modified: 2014-08-12 13:01
category: flapjacks
author: Brian Cobb
---

Brett posed a question in DevChat yesterday:

> Would a language that has better concurrency than Ruby be a better fit for [writing a web service that does some work in response to a `POST` and `POST`s the result back to another service]?

Here are some things I think about when thinking about this question:

1.  **What aspects of the problem can be parallelized, and what aspects require concurrent access to a shared resource?**

    In Supermarket, we parallelize notification emails, but doing so requires concurrent access to shared resources; namely, redis and postgres.

2.  **What primitives can I use to accomplish this task?**

    Where by "primitive" I mean "lowest level of abstraction." In Supermarket, we _could_ use "bare" Ruby threads but instead use Sidekiq workers as our primitive.

3.  **What are the characteristics of a given primitive?**

    One way to compare, say, "bare" Ruby threads with Sidekiq workers is that when Sidekiq workers crash, the failure is isolated to just that worker (the process stays alive) and there's a framework in place to report those crashes. This sort of failure-safety and failure-detection is possible when using Ruby threads as primitives, but requires work on the developer's part.

    We can also compare characteristics to across languages/frameworks. You may have heard goroutines referred to as "lightweight threads." This is because (unlike Ruby) N goroutines do not correspond to N OS threads. Instead, Go manages the details of threads and scheduling, and allows developers to work one level of abstraction higher with a construct that is cheaper (memory-wise) than an OS thread. This is what enables a single Go web service to hold a zillion connections open.x

    One other characteristic to consider is whether the primitive can recover work after a crash. Sidekiq workers push failed jobs onto a special queue and retry the job at a later time. A thread working on a job that was queued in-memory and not tracked anywhere will not retry that job if it crashes unless the developer builds a facility for it to do so.

    I'm sure there are more, but I'm running out of steam and could use some coffee.

This is a big topic, but once we've used the first question to think critically about how our problem breaks down into concurrent/parallel components, we can use the second and third questions to evaluate our options for solving the problem.

---

For further consideration:

1.  [Working With Ruby Threads][1]
2.  [Concurrency is not parallelism (talk by Rob Pike)][2]
3.  [Java Concurrency in Practice][3] (I haven't read this one, but it's been recommended by a bunch of esteemed folks).

[1]: http://www.jstorimer.com/products/working-with-ruby-threads
[2]: http://blog.golang.org/concurrency-is-not-parallelism
[3]: http://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601
