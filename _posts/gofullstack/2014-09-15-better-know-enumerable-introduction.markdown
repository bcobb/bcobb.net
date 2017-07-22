---
layout: post
title: "Better Know Enumerable: Introduction"
seo:
  title: "Better Know Enumerable: Introduction"
slug: better-know-enumerable-introduction
date: 2014-09-17 08:20
category: elsewhere
original: http://gofullstack.com/better-know-enumerable-introduction
author: Brian Cobb
excerpt: >
  Understanding the Enumerable module is essential for any Ruby developer. This
  post kicks off a series which will explore Enumerable's capabilities and
  practical uses.
tags:
  - Better Know Enumerable
  - Programming
  - Ruby
---

Ruby's [`Enumerable`](http://ruby-doc.org/core-2.1.2/Enumerable.html) module is the backbone that gives `Array`, `Hash`, `IO`, and a handful of other core classes methods like `each` and `sort_by`.
More than basic traversal and sorting, it enables a mental and practical shift from imperative to functional programming.
For instance, we _could_ use `each` and a few conditionals to figure out which top-level classes include `Enumerable`:

{% highlight ruby %}
enumerables = []
constant_names = Object.constants

constant_names.each do |constant_name|
  constant = Object.const_get(constant_name)

  if constant.respond_to?(:ancestors)
    if constant.ancestors.include?(Enumerable)
      enumerables << constant
    end
  end
end
{% endhighlight %}

However, using the `map` and `select` methods provided by `Enumerable` break down the problem in a more direct way and without local mutable state.

{% highlight ruby %}
enumerables = Object.constants.
  map { |c| Object.const_get(c) }.
  select { |c| c.respond_to?(:ancestors) }.
  select { |c| c.ancestors.include?(Enumerable) }
{% endhighlight %}

An in-depth exploration of the rest of `Enumerable` could fill a small book, and this post will not attempt to fill in all the gaps.
Instead, it provides a few examples using common and uncommon parts of `Enumerable`'s interface, and hopefully motivates further experimentation with the rest of the module.

---

Despite being used in the proverbially grainy and greyscale example above, `each` is the linchpin of `Enumerable`.
In order for a class to include `Enumerable` it _must_ implement `each`.
For example, `SentenceSearch` below will traverse through all sentences which contain some `word` in a given `body` (for a na&iuml;ve definition of "sentence").
We'll use it to find the word "pudding" in an excerpt from Dickens&rsquo; _Great Expectations_ (text copied from [Project Gutenberg](http://www.gutenberg.org/ebooks/1400)).

{% highlight ruby %}
class SentenceSearch
  include Enumerable

  def initialize(body, word)
    @body = body
    @word = word
  end

  def each
    @body.
      split('.').
      select { |sentence| sentence.include?(@word) }.
      each { |sentence| yield "#{sentence.lstrip}." }
  end
end

excerpt = <<-EXCERPT
  By degrees, I became calm enough to release my grasp
  and partake of pudding. Mr. Pumblechook partook of
  pudding. All partook of pudding. The course terminated,
  and Mr. Pumblechook had begun to beam under the genial
  influence of gin and water. I began to think I should
  get over the day, when my sister said to Joe,
  "Clean plates,--cold."
EXCERPT
usable_excerpt = excerpt.lines.map(&:strip).join(' ')

puddings = SentenceSearch.new(usable_excerpt, 'pudding')
{% endhighlight %}

Being able to traverse through these sentences is useful&mdash;many search engines provide this sort of view when displaying search results&mdash;but `Enumerable` facilitates working with these collections of sentences in a surprising number of ways.

We can find the shortest and longest sentence:

{% highlight ruby %}
pp puddings.minmax_by { |sentence| sentence.length }

# ["All partook of pudding.",
#  "By degrees, I became calm enough to release my grasp and partake of pudding."]
{% endhighlight %}

Or just the shortest sentence:

{% highlight ruby %}
pp puddings.min_by { |sentence| sentence.length }

# "All partook of pudding."
{% endhighlight %}

Using the union operation on `Array` [`|`](http://www.ruby-doc.org/core-2.1.2/Array.html#method-i-7C), we can compile the collection of unique words in all of the sentences:

{% highlight ruby %}
puddings.
  map { |sentence| sentence.split(' ') }.
  reduce { |c, words| c | words }

# ["By",
#  "degrees,",
#  "I",
#  "became",
#  "calm",
#  "enough",
#  "to",
#  "release",
#  "my",
#  "grasp",
#  "and",
#  "partake",
#  "of",
#  "pudding.",
#  "Pumblechook",
#  "partook",
#  "All"]
{% endhighlight %}

`Enumerable` is a generic module for collection traversal; with a little imagination we could conjure up dozens of additional ways to process the base collection of pudding-related sentences.
In future posts, I'll cover `Enumerable`'s methods in greater depth, and show real-world examples of refactoring towards an `Enumerable` class to reap the benefits of its familiar and powerful interface.
