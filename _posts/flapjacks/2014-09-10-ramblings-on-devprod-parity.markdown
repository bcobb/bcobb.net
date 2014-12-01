---
layout: post
title: Ramblings on dev/prod parity
permalink: ramblings-on-devprod-parity
published: true
date: 2014-09-10 13:31
category: flapjacks
author: Brian Cobb
---

I don't know the origin of the term "dev/prod parity," but I think it's fair to say that the [12 Factor App philosophy][1] helped [popularize][2] it. I think it's also fair to say that tools like Vagrant, Docker, and myriad configuration management frameworks sprung out of a desire to achieve such parity, and their development has had a snowball effect of simultaneously spreading the idea that dev-prod parity is useful and lowering the bar to attaining some level of parity.

Sadly, I have yet to see in practice an app that combines a tool like Vagrant with a tool like Chef to effortlessly run a near-production setup locally for the purpose of development. And most writing on the topic reminds of this tired "draw an owl" meme:

![a helpful guide on drawing owls][3]

Here's how I would like to break down the steps to dev-prod parity, at least for Ruby apps running on Chef-configured clusters:

1.  Carve production into "public" recipes by the services exposed in the notional cluster. For a Rails app that runs jobs with Sidekiq, this might look like: web, worker, postgresql, redis.
2.  A development box is simply a node whose run list includes all of those recipes, and whose data bag configures the recipe to be development-friendly:
  * Rails should be configured to reload code
  * "deploys" happen by syncing files on the host the files on the guest automatically
  * bundler installs all gems so that we can take our existing testing workflow and wrap it in `vagrant ssh -c` commands
  * probably some other stuff

3.  A production cluster is a set of nodes (possibly just one node) that are configured for production (e.g. deploy with git and automatically run migrations).

My hope is that this minimizes or eliminates the so-called "tool gap" between development and production. If development "stands up," we can be reasonably sure production will stand up. Development will use the same process supervisor, the same init scripts, the same configuration mechanism as production, so we can even get a mini-prod running locally by configuring everything except the code delivery strategy to be production-esque.

But after a day of fighting with Vagrant and VMWare, I mostly just want to quietly introduce foreman for local development and move on with my life.

 [1]: http://12factor.net/
 [2]: http://12factor.net/dev-prod-parity
 [3]: http://f.cl.ly/items/2j2v2t3N3A1V3N080e2z/1395294474939.jpg
