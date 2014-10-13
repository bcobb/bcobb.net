---
layout: post
title: "Hacker School Read Along: On Understanding Data Abstraction"
permalink: hacker-school-read-along-on-understanding-data-abstraction
category: posts
date: 2014-10-12 11:33
---

This week's Hacker School Paper of the Week is [William Cook](http://www.cs.utexas.edu/~wcook/)'s [_On Understanding Data Abstraction, Revisited_](http://www.cs.utexas.edu/~wcook/Drafts/2009/essay.pdf).
The premise is a question: "what is the relationship between _objects_ and _abstract data types_?"
Cook argues that the typical textbook answer---"objects are a kind of abstract data type"---is incorrect.
While both objects and abstract data types are forms of _data abstraction_, they are "fundamentally different and in many ways complementary."
The paper is an exploration of the terms "object" and "abstract data type," and an attempt to highlight the fundamental differences between the two, in part by showing an implementation of a Set as an ADT, and as a collection of objects.

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

### Further Thoughts

I've often read something along the lines of "OO code results in leaky abstractions" but &#167;3.5 presents concrete examples of how the OO Set will leak if we want to optimize the `union` method, or implement an `intersect` method.
Michael Feathers has done a bit of work in analyzing git repositories for how code changes over time. This makes me wonder how prevalent this sort of interface creep is, and what it means for, say, the maintainability of a codebase (whatever that means).

I'm struggling to see the distinction between ADTs and "pure" OO interfaces as clearly as Cook sees it (I really want to, though).
The example of a "bad object" seems contrived in the sense that it's not clear how ADTs protect against this sort of error, deliberate or not.
Is there an implicit assumption that testing ADTs necessarily involves writing generative tests, which would inevitably squash these sorts of bugs?

The discussion on "simple" and "complex" operations in &#167;4.2 feels like the most fundamental distinction---that pure OO programming does not support complex operations, whereas operations on ADTs can be complex--but is it possible to formalize OO in such a way that allows complex operations?
If so, is it necessarily true that objects and type abstraction are orthogonal?
Is it even worth trying to do this, given that ADTs already have a rich theoretical underpinnings?
Maybe this is what folks mean when they talk about OO ultimately being a dead-end.

In any case, I think this paper (perhaps obviously) has a lot of overlap with Gary Bernhardt's [Functional Core, Imperative Shell](https://www.destroyallsoftware.com/talks/boundaries).

### Highlights

#### &#167;2.6

> [ADTs have a] solid connection to mathematics.
An ADT has the same form as an abstract algebra: a type name representing an abstract set of values together with operations on the values.
The operations can be unary, binary, multi-ary, or nullary (that is, constructors) and they are all treated uniformly.

#### &#167;3.1

> Object interfaces are essentially higher-order types, in the same sense that passing functions as values is higher-order.
Any time an object is passed as a value, or returned as a value, the object-oriented program is passing functions as values and returning functions as values.
The fact that the functions are collected into records and called methods is irrelevant.
As a result, the typical object-oriented program makes far more use of higher-order values than many functional programs.

#### &#167;3.8

> A common complaint is that it is impossible to determine what code will execute when invoking a method.
This is no different from common uses of first-class functions.
If this objection is taken seriously, then similar complaints must be leveled against ML and Haskell, because it is impossible (in general) to determine what code will run when invoking a function value.

and:

> Object-oriented programming is designed to be as flexible as possible.
It is almost as if it were designed to be as difficult to verify as possible.

#### &#167;4.3

> Abstract data types define operations that collect together the behaviors for a given action.
Objects organize the matrix the other way, collecting together all the actions associated with a given representation.
It is easier to add new operations.

#### &#167;5.4

> One conclusion you could draw from this analysis is that the untyuped lambda-calculus was the first object-oriented language. (&#167;5.4)

