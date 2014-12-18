---
layout: post
title: "Gratuitous Ruby: Counting Item Frequency"
seo:
  title: "Gratuitous Ruby: Counting Item Frequency"
published: true
permalink: gratuitous-ruby-counting-item-frequency
date: 2014-12-08 22:41
category: misc
---

Tonight on Twitter, [@jessitron] posted the following:

<blockquote class="twitter-tweet" lang="en"><p>Counting in <a href="https://twitter.com/hashtag/Ruby?src=hash">#Ruby</a>:&#10;[&quot;a&quot;,&quot;b&quot;,&quot;a&quot;].inject(<a href="http://t.co/42s0B3GL9E">http://t.co/42s0B3GL9E</a>(0)) { |m,o| m[o] += 1; m }&#10; =&gt; {&quot;a&quot;=&gt;2, &quot;b&quot;=&gt;1}&#10;<a href="https://twitter.com/mattruzicka">@mattruzicka</a> <a href="https://twitter.com/hashtag/STLRuby?src=hash">#STLRuby</a></p>&mdash; Jessica Kerr (@jessitron) <a href="https://twitter.com/jessitron/status/542150589728440320">December 9, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

The other week, I needed to do exactly this, and came up with a slightly different approach.
Here's my (gently elaborated) version of Jessica's snippet:

```ruby
%w(a b a).reduce(Hash.new) do |map, item|
  map.merge(item => 1) do |key, sum, increment|
    sum + increment
  end
end
```

I don't know that it's better or worse, but it has two properties I like:

1. The body of `reduce` is immutable.
1. It utilizes `merge`'s ability to take a block to resolve conflicting updates.[^1]

The last point is the linchpin of the block.
When we merge in a new `item` key into `map`, its value is set to `1`.
If we encounter that key again, `merge` sets the value `key` to the result of its block.
The block itself takes three arguments: the conflicting key, the existing value of that key, and the value we attempted to merge.
For our purposes, the key isn't necessary; but notice that its existing value is simply the number of `item` keys we've tried to merge in the past (starting with `1`), and the value we're attempting to merge is the value by which we increment the count.

Anyway, there's not really a point to this post, other than that it can be fun to fart around with Enumerable on a sleety Monday night.

[@jessitron]: https://twitter.com/jessitron

[^1]: My friend Ransom pointed out that since `merge` performs a copy of the source Hash, the code I've written has polynomial complexity (traversal &times; merge). There's no practical downside to changing it to `merge!` which would avoid the copy.
