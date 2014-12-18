---
layout: post
title: "Better Know Enumerable: Partition"
seo:
  title: "Better Know Enumerable: Partition"
permalink: better-know-enumerable-partition
date: 2014-10-03 09:04
category: elsewhere
author: Brian Cobb
original: http://gofullstack.com/better-know-enumerable-partition
excerpt: >
  Understanding the Enumerable module is essential for any Ruby developer. This
  post shows a case where its partition method turns tangled, imperative code
  into direct, functional code.
tags:
  - Better Know Enumerable
  - Programming
  - Ruby
---

In the [introduction to Better Know Enumerable](/better-know-enumerable-introduction), I claimed that `Enumerable` enables a shift from imperative to functional programming.
In this post, I'd like to demonstrate this sort of shift by using the `partiton` method to replace imperatively splitting a collection into two complementary parts.

The documentation for `partition` shows a contrived example:

```ruby
(1..6).partition { |v| v.even? } #=> [[2, 4, 6], [1, 2, 3]]
```

An imperative solution might look like this:

```ruby
left = []
right = []

(1..6).each { |v| v.even? ? left.push(v) : right.push(v) }

[left, right]
```

Using `partition` is more succinct, but it's not always so obvious that it's _possible_ to tackle a problem with `partition`.
Here's an example adapted from a recent project where we were able to identify such a refactoring.

Suppose we're purveyors of fine puddings, and we've modeled our inventory in the following way[^1]:

```ruby
class Pudding < Struct.new(:flavor, :size)
  STOCKED_FLAVORS = %w(vanilla chocolate)
  STOCKED_SIZES = %w(cup double-cup)

  def valid?
    STOCKED_FLAVORS.include?(flavor) && STOCKED_SIZES.include?(size)
  end
end
```

We've just gotten a shipment; the first six lines of the manifest CSV look like this:

```
vanilla,cup
chocolate,double-cup
butterscotch,cup
vanilla,double-cup
chocolate,party-bowl
```

Sadly, it appears that our pudding provider has shipped us some puddings that we don't stock.
We need to generate a packing slip for the return shipment as we process the shipment:

```ruby
require 'csv'

puddings = CSV.foreach('/path/to/manifest.csv').map do |flavor, size|
  Pudding.new(flavor, size)
end

def process(pudding)
  # some cool pudding processing
end

CSV.open('/path/to/return.csv', 'wb') do |csv|
  puddings.each do |pudding|
    if pudding.valid?
      process(pudding)
    else
      csv << [pudding.flavor, pudding.size]
    end
  end
end
```

The conditional inside `each` makes me think there's an opportunity to use `partition` to make this code a succession of smaller and more declarative chunks of work:

```ruby
require 'csv'

puddings = CSV.foreach('/path/to/manifest.csv').map do |flavor, size|
  Pudding.new(flavor, size)
end

def process(pudding)
  # some cool pudding processing
end

stock, send_back = puddings.partition(&:valid?)

stock.each { |pudding| process(pudding) }

CSV.open('/path/to/return.csv', 'wb') do |csv|
  send_back.each do |pudding|
    csv << [pudding.flavor, pudding.size]
  end
end
```

I like the second approach quite a bit better.
Each step in the process is more isolated and concise than it was before.
To work with either the pudding processing or the return slip generation is to _not_ work with any other component in the system.
In my experience, this indicates that we've arrived at a better design.
As a bonus, having the collection of stockable puddings in one array and the collection of puddings to return in another makes it easy to compute metadata on the shipment. For example, here's the flavor that was shipped in the highest quantity:

```ruby
best_flavor, _ = stock.
  group_by(&:flavor).
  sort_by { |_, puddings| puddings.count }.
  last

best_flavor == "vanilla"
```

`partition` is a nifty way to turn imperative code into declarative code.
Keep an eye out for `each` blocks with conditionals; when you find one, refactor it to use `partition` and see how it feels!

[^1]: The discussion of whether to inherit from `Struct` or to set the value of a constant to an instance of `Struct` is beyond the scope of this post
