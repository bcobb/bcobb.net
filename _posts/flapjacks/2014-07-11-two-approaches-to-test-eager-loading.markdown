---
layout: post
title: Two approaches to test eager loading
seo:
  title: Two approaches to test eager loading
permalink: two-approaches-to-test-eager-loading
published: true
date: 2014-07-11 16:22
category: flapjacks
author: Brian Cobb
---

We've been updating some queries in Supermarket to take advantage of ActiveRecord's eager loading. In talking with [Brett](http://brettchalupa.com) about the when, why, and how of eager loading, I wrote a gist of two ways one can write tests around eager loading (it *is* a change in behavior, after all). This post will borrow from that gist and expand upon the motivation to use either.

* * *

As an aside, if you're not familiar, *eager loading* means that instead of just fetching, say, a list of Posts, and then fetching each Post's author individually whenever we need to display the post's author, we fetch a list of Posts *and* each Post's author *and* associate the results appropriately all in one shot. The term "N+1 bug" refers to the first behavior, where if our original query returns N posts, and we work with each post's author, then for each post we would issue one additional query: a total of N+1 queries. If we instead eager load authors, we only issue 2 queries.

* * *

The first approach is a direct test. Recall from the aside: when we *don't* eager load Authors, we issue one query per post author. That is:

```ruby
posts = Post.all.to_a # one query, returns an array of Posts
posts[0].author # one query, returns an Author if it exists, nil otherwise
posts[1].author # one query, returns an Author if it exists, nil otherwise
# etc
```


One can imagine a parallel, cruel universe where Author records are frequently deleted without warning, causing our `posts[i].author` lines above to sometimes (and unexpectedly) return `nil`. As programmers, we can create a facsimile of such a universe, and use it to verify that we've eager loaded authors:

```ruby
posts = Post.includes(:author).all.to_a # the includes is the secret sauce

Author.delete_all # evil is afoot!

posts[0].author # no query, returns an Author if it existed for eager loading, nil otherwise
posts[1].author # no query, returns an Author if it existed for eager loading, nil otherwise
```

An actual test might look something like this:

```ruby
it 'eager loads authors' do
  2.times do
    Post.create!(author: Author.create!)
  end

  posts = Post.scope_which_eager_loads_authors.to_a

  Author.delete_all

  expect(posts.all?(&:author)).to be_true
end
```

To me, a test like this implies that we're eager loading because we care about *determinism*. Whether or not a given `post` in `posts` has an author is determined when we retrieve the list of posts. That is, the *motivation* for eager loading is because we want our code to behave deterministically given some state of the universe, and we write our test to reflect as much.

* * *

A different motivation for eager loading records is for performance. Generally speaking, it is at least an order of magnitude slower to issue N queries for one row than it is to return N rows with a single query (there are, of course, exceptions). The example given above is also a canonical example of when eager loading improves performance, and we can write a test that guides the reader to this conclusion:

```ruby
it 'eager loads authors to improve performance' do
  10.times do
    Post.create!(author: Author.create!)
  end

  eager_loaded = Post.scope_which_eager_loads_authors.to_a
  lazy_loaded = Post.all.to_a

  eager_loaded_duration = Benchmark.realtime do
    eager_loaded.each(&:author)
  end

  lazy_loaded_duration = Benchmark.realtime do
    lazy_loaded.each(&:author)
  end

  # eager loading should be an order of magnitude faster than lazy loading
  expect((eager_loaded_duration * 10) < lazy_loaded_duration).to be_true
end
```

* * *

Using one or both of these approaches helps to communicate the original motivation for introducing eager loading, and protects future programmers from accidentally introducing a regression.
