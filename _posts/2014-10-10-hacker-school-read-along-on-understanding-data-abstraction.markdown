---
layout: post
permalink: hacker-school-read-along-on-understanding-data-abstraction
category: posts
title: "Hacker School Read Along: On Understanding Data Abstraction"
published: false
---

This week's Hacker School Paper of the Week is [William Cook](http://www.cs.utexas.edu/~wcook/)'s [_On Understanding Data Abstraction, Revisited_](http://www.cs.utexas.edu/~wcook/Drafts/2009/essay.pdf).
The premise is a question: "what is the relationship between _objects_ and _abstract data types_?"
Cook argues that the typical textbook answer---"objects are a kind of abstract data type"---is incorrect.
While both objects and abstract data types are forms of _data abstraction_, they are "fundamentally different and in many ways complementary."
The paper is an exploration of the terms "object" and "abstract data type," and an attempt to highlight the fundamental differences between the two by showing an implementation of a Set as an ADT, and as a collection of objects.

Though the paper dovetails nicely with my progress through SICP (especially [Section 2.1.3](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-14.html#%_sec_2.1.3)), I still feel like I'm missing an important piece of the distinction.
More on that confusion after the code.

### Data Abstraction in Ruby

Here is a slightly wacky Ruby implementation of the Set-as-objects shown in the paper, one which does not expose the notion of a class to its users.
I wanted to emphasize not only that the notion of a class is not important, but also demonstrate that it's possible to abstract over data without using instance variables.
This code, along with a version that does use named classes, is [on GitHub](https://github.com/bcobb/on_understanding_data_abstraction_revisited-ruby).

```ruby
def EmptySet
  Class.new do
    define_method(:empty?) { true }

    define_method(:contains?) { |i| false }

    define_method(:insert) { |i| InsertSet(self, i) }

    define_method(:union) { |s| s }
  end.new
end

def InsertSet(s, n)
  if s.contains?(n)
    s
  else
    Class.new do
      define_method(:empty?) { false }

      define_method(:contains?) do |i|
        i == n || s.contains?(i)
      end

      define_method(:insert) do |i|
        InsertSet(self, i)
      end

      define_method(:union) do |s|
        UnionSet(self, s)
      end
    end.new
  end
end

def UnionSet(s1, s2)
  Class.new do
    define_method(:empty?) do
      s1.empty? && s2.empty?
    end

    define_method(:contains?) do |i|
      s1.contains?(i) || s2.contains?(i)
    end

    define_method(:insert) do |i|
      InsertSet(self, i)
    end

    define_method(:union) do |s|
      UnionSet(self, s)
    end
  end.new
end

set = UnionSet(
  EmptySet().insert(1).insert(2),
  UnionSet(
    InsertSet(EmptySet(), 3),
    InsertSet(EmptySet(), 4)
  )
)

(1..5).select { |i| set.contains?(i) }
# => [1, 2, 3, 4]
set.empty?
# => false
```

### Confusion

As I said above, I'm still struggling to see the distinction between ADTs and "pure" OO interfaces as clearly as Cook sees it.

