---
layout: post
title: The Idempotency Trap
seo:
  title: The Idempotency Trap
permalink: the-idempotency-trap
published: true
date: 2014-07-01 14:02
category: flapjacks
author: Brian Cobb
---

[I just fell into the idempotency trap][1]!

As you may or may not know, one reason HTTP methods exist (`GET`, `POST`, `PUT`, and `DELETE` being the most prevalent) is because different HTTP methods signal different expectations with regard to what sorts of side-effects a request will cause. `GET`, `PUT`, and `DELETE` are *idempotent*, which is to say that issuing the same `GET`, `PUT`, or `DELETE` request a jillionty times in a row should have the same observable side-effects as issuing it once. `GET` is special in that it should be *safe*: it should not generate any side-effects observable to the client (if this is the case, it is trivially idempotent).

The **idempotency trap** is when one uses `GET` as a primary frame of reference for what constitutes idempotency, and confuses *safety* with *idempotency*. In the case of the linked PR comment, were an admin to issue the same request to combine two organizations twice, the second one *would* fail, but that doesn't mean it's not idempotent! It just means it's not safe. Reading through the `OrganizationController`'s `combine` method suffices to verify that it is indeed idempotent.

I'd recommend *not* using `GET` as a frame of reference for examples of idempotency. Instead, use `DELETE`. It's not too surprising that deleting, say, the same Checklist item twice may cause an error, but it's clear that in a well-behaved system, issuing two such requests would *only* delete that one item.

To close, I should emphasize that these method types are merely signals. Just because a server implements a given endpoint using an idempotent HTTP method makes no guarantees about its idempotency or safety. Just because there's a URL that responds to `DELETE` requests does not mean that anything's being deleted. Cruel and/or misinformed server developers can do whatever they want.

 [1]: https://github.com/opscode/supermarket/pull/521/files#discussion_r14403201
