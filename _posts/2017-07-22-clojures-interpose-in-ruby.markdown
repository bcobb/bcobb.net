---
layout: post
title: "Clojure's interpose in Ruby"
seo:
  title: "Clojure's interpose in Ruby"
  description: "Implementing interpose in Ruby using Enumerable"
published: true
slug: clojures-interpose-in-ruby
category: tbd
date: 2017-07-22 12:59
modified: 2017-08-03 10:00
---

[`interpose`][interpose] is one of many things I miss about Clojure when I'm writing Ruby.
Its behavior is kind of like if `Array#join` had a cousin that didn't return a `String`, but instead returned another `Array`:

```clojure
(interpose :sep [])
;; ()

(interpose :sep [1])
;; (1)

(interpose :sep [1 2])
;; (1 :sep 2)
```

It's not something I need terribly often, but it's really nice to have when I do.
For instance, if I've got a collection of objects which represent some workflow, and I want to gate each step in the workflow with a confirmation step, `interpose` is super handy:

```clojure
(def workflow [info step-a step-b step-c])

(def gated-workflow (interpose confirm-next workflow))

gated-workflow
;; (info confirm-next step-a confirm-next step-b confirm-next step-c)
```

Luckily, Ruby makes it easy to open up `Enumerable` and define our own.
Let's start by reading the doc string for `interpose`:

> `(interpose sep)` `(interpose sep coll)`
>
> Returns a lazy seq of the elements of coll separated by sep.
>
> Returns a stateful transducer when no collection is provided.

That second line isn't really pertinent to us, but the first line makes me think we'll want to return an `Enumerator`, so let's start there.

In Clojure, printing a lazy sequence [realizes it], but the same is not true for Ruby `Enumerator` objects.
This means that to test out our `interpose` in IRB and see the results, we'll need to convert it to an `Array`.
This is what it'll look like when we're done:

```ruby
[].interpose(:sep).to_a
# []

[1].interpose(:sep).to_a
# [1]

[1, 2].interpose(:sep).to_a
# [1, :sep, 2]
```

For a first stab, let's just iterate through the underlying collection, and shovel each entry, along with the separator, to our `Enumerator`'s [yielder]:

```ruby
module Enumerable
  def interpose(sep)
    Enumerator.new do |y|
      each do |item|
        y << item
        y << sep
      end
    end
  end
end
```

```ruby
[].interpose(:sep).to_a
# []

[1].interpose(:sep).to_a
# [1, :sep]

[1, 2].interpose(:sep).to_a
# [1, :sep, 2, :sep]
```

This isn't quite what we want; the base case of an empty collection works, but the other cases have one too many separators.
It's a good start, though, and we can tweak our use of `Enumerable#each` so that it returns its own `Enumerator`.
This will still allow us to iterate through the underlying collection one element at a time, but with the added ability to detect when we've reached the end of the collection.

Here's a version of `interpose` that behaves the same as the last one, but which uses `Enumerator#next` to step through the underlying collection.
`Enumerator#next` raises `StopIteration` if it would advance beyond the end of the underlying collection, so we'll catch that and break out of the loop when it's raised.

```ruby
module Enumerable
  def interpose(sep)
    Enumerator.new do |y|
      items = each

      loop do
        begin
          y << items.next
        rescue StopIteration
          break
        else
          y << sep
        end
      end
    end
  end
end
```

To get rid of the extra separator at the end of the collection, we need to detect if the element we just visited with `Enumerable#next` was the last element of the collection.
Luckily, `Enumerator` implements a `peek` method that will return the next element _or_ raise `StopIteration` if the `Enumerator` is at the end of the collection.
Crucially, `peek` differs from `next` in that it does not advance the `Enumerator`.
If `Enumerator#peek` succeeds, we know to shovel the separator into the yielder.
If it fails, we're done iterposing:

```ruby
module Enumerable
  def interpose(sep)
    Enumerator.new do |y|
      items = each

      loop do
        begin
          y << items.next
        rescue StopIteration
          break
        end

        begin
          items.peek
        rescue StopIteration
          break
        else
          y << sep
        end
      end
    end
  end
end
```

Success!

```ruby
[].interpose(:sep).to_a
# []

[1].interpose(:sep).to_a
# [1]

[1, 2].interpose(:sep).to_a
# [1, :sep, 2]
```

Here's `Array#join` built on top of `interpose`:

```ruby
class Array
  def join(sep)
    interpose(sep).each_with_object("") do |segment, string|
      string << String(segment)
    end
  end
end
```

(Don't actually do this).

And because it's a method on `Enumerable`, it can be used pretty pervasively:

```ruby
{one: 1, two: 2, three: 3}.interpose(:sep).to_a
# [[:one, 1], :sep, [:two, 2], :sep, [:three, 3]]

StringIO.new("line one\nline two\nline 3\n").interpose("|").to_a
# ["line one\n", "|", "line two\n", "|", "line 3\n"]
```

---

This isn't something I'm racing to introduce at work, but it was a fun exercise in using `Enumerable` and `Enumerator` to bring a useful function in Clojure into Ruby.
It also led me to wonder what it would take to extend the core Ruby language to support `interpose`.
I've never really written C, but I intend to explore this question some more, and will hopefully have something to show for it soon!

---

### Addendum, 2017-08-03

Avdi Grimm(!) wrote a [follow-up post][riffing], riffing on this idea.
His implementation is not only significantly faster, but also a bit more Rubyish -- it allows the caller of `interpose` to pass an optional block which receives items and separators as its sole argument.
He also drives out the behavior using tests, using RSpec's shared example groups to capture shared behavior between the block and block-less variants.

Thanks, Avdi!

[interpose]: https://clojuredocs.org/clojure.core/interpose
[enumerator docs]: https://ruby-doc.org/core-2.4.0/Enumerator.html
[yielder]: https://ruby-doc.org/core-2.4.0/Enumerator.html#method-c-new
[realizes it]: http://clojure-doc.org/articles/language/laziness.html#realizing-lazy-sequences-forcing-evaluation
[riffing]: http://www.virtuouscode.com/2017/08/02/riffing-on-interpose-in-ruby/
