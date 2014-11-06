---
layout: post
title: "Curry: How Chef Software (Mostly) Automated Their CLA"
permalink: curry-how-chef-software-mostly-automated-their-cla
date: 2014-10-24 12:01
category: elsewhere
author: Brian Cobb
original: http://gofullstack.com/curry-how-chef-software-mostly-automated-their-cla
excerpt: >
  To replace the tedious and manual process of verifying that contributors to Chef's Open Source projects were covered by a CLA, FullStack built Curry, a GitHub bot that interfaces with Supermarket to automate CLA-coverage verification.
tags:
  - Projects
  - Programming
  - Ruby
  - Supermarket
---

Before the launch of Supermarket, Chef Software, Inc. manually verified that each contribution to its open source software had been authored by someone who had either signed their Individual Contributor License Agreement (ICLA), or who was covered under a company's Corporate Contributor License Agreement (CCLA).
When FullStack took over the Supermarket project, our first task was to automate this process as much as possible.

At a high level, there are three distinct parts:

- It must be possible for individuals to sign an ICLA, and for companies to sign a CCLA and manage the list of contributors it covers.
- Before accepting a contribution to their open source software, Chef needs to determine that the contribution is authored by folks who are covered by a CLA.
- When the authors of a contribution that was once-unauthorized are all covered by the CLA, Chef needs to know that the contribution is eligible to be accepted.

Originally, the first bullet point was handled by an EchoSign form, and the last two bullet points by a variety of spreadsheet-like records and manual checks.
By all accounts, it was a tedious and confusing process for everyone involved.

These days, Supermarket handles all three with a combination of two clean forms to take signatures, and a small (but tasty) GitHub monitor named Curry.

### How Curry Works

Before Curry can do any work, an admin must use its small web interface to add a GitHub repository.
Following this, Curry subscribes to that repository's PubSubHubbub[^1] hub for Pull Request-related events.[^2]
For the subset of these events that correspond to new or updated Pull Requests, Curry will figure out if the contribution is covered by a CLA, and make a note on the Pull Request accordingly.

The incoming event contains enough data to identify the associated Pull Request, but it doesn't have the good stuff: the list of commit authors on the Pull Request.
Even if it did, we eventually want to communicate the state of the Pull Request _back_ to the authors and to the folks at Chef, and that's going to require we use GitHub's robust, but rate-limited API.
No matter which way we slice it, we're going to be making several API calls, some of which might need to wait an indeterminite period for a rate-limit to clear.

This is a perfect fit for a Sidekiq worker!
When Curry receives a PubSubHubbub event, it logs the event, and pushes a job onto the `ImportPullRequestCommitAuthorsWorker` worker's queue.

This worker, as the name implies, imports the commit authors on the Pull Request as database records.
The very last thing this worker does is push a job onto the `ClaValidationWorker` worker's queue.
(As an aside, the names of our workers are specific to the point of absurdity. Is this normal?)
The `ClaValidationWorker` is responsible for validating the CLA status of the Pull Request in question, and for communicating that status by updating the Pull Request on GitHub.
In the event that a commit author on the Pull Request is not authorized, Curry will attempt to inform that user that they need to be covered by a CLA, and will point them to Supermarket.

Ideally, this means that at some point the unauthorized commit authors will become covered by the CLA.
When they do, all of their outstanding Pull Requests will be re-evaluated by the `PullRequestAppraiserWorker` worker, which simply queues jobs for the `ClaValidationWorker` worker.

With these two major pieces in place, Curry will keep tabs on open Pull Requests and on newly-authorized Supermarket users to automate what was once a laborious process.
To get a sense for the nitty-gritty of how Curry works, its code is nearly self-contained in the Supermarket codebase.
Consider starting at its [controllers](https://github.com/opscode/supermarket/tree/79ea1aa2c48913bd0a94600add8a2b079c3b70c7/app/controllers/curry), then following the job queue to the [workers](https://github.com/opscode/supermarket/tree/79ea1aa2c48913bd0a94600add8a2b079c3b70c7/app/workers/curry), and end up in the [models](https://github.com/opscode/supermarket/tree/79ea1aa2c48913bd0a94600add8a2b079c3b70c7/app/models/curry) in `ImportPullRequestCommitAuthors`, `PullRequestAnnotator`, and `RepositorySubscriber`.

### The Future of Curry

One scenario that Curry does not currently automate is when a contributor authors a commit with an email address that isn't verified by GitHub.
For instance, when my colleague Brett and I pair, we use my FullStack email address, but with an alias to indicate that Brett worked with me.
This email address belongs to _both_ of us, really, so I haven't added it as an email address on GitHub.
However, if GitHub hasn't verified the email address, then the commits Curry sees through GitHub's API will not be attributed to a GitHub user, and will thus appear to not be covered by a CLA.
It'd be awesome if Supermarket had a feature to verify email addresses so that Curry could authorize these commits.[^3]

Seth Vargo [pointed out](https://github.com/opscode/supermarket/issues/575) that another nice-to-have would be some insight into stale Pull Requests: which PRs are not authorized, and have not been updated since Curry left a comment.

### The Wrath of Curry

An innocuous change to GitHub's PubSubHubbub events caused Curry to break several Pull Request pages.
Fortunately, this did not impede critical contributions from being merged, and the incident contained a valuable lesson in the utility of whitelisting instead of blacklisting.

[The facts are these](http://tvtropes.org/pmwiki/pmwiki.php/Series/PushingDaisies?from=Main.PushingDaisies):

1. From the time development started on Curry until 2014-07-29, there were four possible Pull Request-related PubSubHubbub events: opened, closed, reopened, or synchronized.
1. Curry's PubSubHubbub callback as of 2014-07-29 _only_ ignored the "closed" event.
1. Curry automates the process of adding a label to indicate a Pull Request is covered by a CLA, and removing that label if a Pull Request ceases to be authorized.
1. Curry's initial implementation _always_ removed the existing label before doing anything else, so as to reset the state of the world.
1. On 2014-07-29, GitHub rolled out several new Pull Request-related PubSubHubbub events, including "labeled" and "unlabeled."

Following the last change, any new, authorized Pull Request would cause Curry to add a label, then receive a PubSubHubbub event for that addition, then remove the label; then, it'd receive a PubSubHubbub event for that _subtraction_, and so on.
The critical error was that we (implicitly) assumed that Curry itself would never trigger PubSubHubbub events, and baked this assumption into the app with what was effectively a blacklist of one event ("closed").
To amend this, [Brett updated the PubSubHubbub callback](https://github.com/opscode/supermarket/commit/9204c7f235b6645c8bd3d5b64a781b0adf30d3f4#diff-1) to only act on a _whitelist_ of actions.
Once that change had been deployed (and once the fine folks at GitHub had cleaned up the Unicorn-timeout-inducing data), balance was restored to the Supermarket.

[^1]: [PubSubHubbub](http://en.wikipedia.org/wiki/PubSubHubbub), if you're not familiar, is a distributed publish-subscribe protocol. In practice, it means that Curry can tell GitHub "hey, send a POST to this URL whenever something happens with the Pull Requests in this repository" and GitHub will do just that.
[^2]: As a bonus, after a repository is added to Curry, a worker iterates through all of the repo's open PRs and annotates it appropriately.
[^3]: Supermarket did in fact have this capability early in the project. However, for reasons I cannot remember, we scrapped it before launch.
