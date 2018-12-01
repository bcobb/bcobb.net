---
layout: post
title: Refactoring towards ease by implementing Enumerable
seo:
  title: Refactoring towards ease by implementing Enumerable
  description: A description of a refactoring using Ruby's Enumerable module
slug: refactoring-towards-ease-by-implementing-enumerable
published: true
date: 2014-05-13 15:44
modified: 2014-05-13 15:44
category: flapjacks
author: Brian Cobb
---

The Supermarket project not only replaces the existing Opscode Community Site codebase, but also switches its database from MySQL to PostgreSQL. As such, one of our pre-launch tasks is to devise a [data migration script][1] that we can run to import all relevant Community Site data into Supermarket. We'll also use this script to keep Supermarket up-to-date with the Community Site during the soft launch period, where they'll run in parallel.

This post details a refactoring in the data migration codebase, which was motivated by wanting a more granular way to diagnose records whose migrations raised an exception. I might also contend that this refactoring is motivated by valuing [ease and joy at work][2]. Regardless, the gist is that we take what you might call a [Method Object][3] and have it instead implement `Enumerable` for great good.

At a high level, our data migration looks like this:

```ruby
records_which_should_be_imported.each do |record|
  importer = SomeImporter.new(record)

  ActiveRecord::Base.transaction do
    begin
      importer.call
    rescue
      raise ActiveRecord::Rollback
    end
  end
end
```

Where `SomeImporter.new(record)` attempts to establish the state required to perform an import, and `importer.call` uses that state to build and save Supermarket's object graph. We wrap `importer.call` in an ActiveRecord transaction so that we don't leave the database in a messy state if the import fails.

It's worth noting that these `importer` objects grew out of a similar desire to debug with ease, but are not in practice all that easy to use. If we have a `record` which causes `importer` to raise an exception, we need to do some gymnastics to get at the invalid object:

```ruby
error = nil

begin
  SomeImporter.new(record).call
rescue => e
  error = e
end

error.record # and only if the error is an ActiveRecord error
```

Given the relatively poor state of the data on the existing Community Site, it quickly becomes desirable to cut to the proverbial chase. In particular, we care about three things when importing a record fails:

1.  **Have we imported all of the data upon which this record depends?**

    For example, we want to skip a cookbook record if we hadn't already imported that cookbook's owner.

2.  **Which records will we attempt to save in Supermarket, if any?**

    Some importers, such as the Category importer, just save one record in Supermarket. Others, such as the Cookbook importer, save at least two records. Those two importers only create *new* records in Supermarket; others update *existing* records.

3.  **What happens when we try to save each record in Supermarket?**

    In the case of Cookbooks failing to import, for example, it's important to know whether the Cookbook data is invalid or whether its associated CookbookVersion records are invalid.

If we shift our approach so that `importer` implements `Enumerable` to iterate over the records it deems need to be saved *without* saving them, we can answer all of these questions with ease. So, where an importer's `call` method may have once looked like this:

```ruby
def call
  something = Something.new
  something.save!
end
```

We rename it to `each` and write it like this:

```ruby
def each
  yield ::Something.new
end
```

Our import now looks like this:

```ruby
records_which_should_be_imported.each do |record|
  importer = SomeImporter.new(record)

  ActiveRecord::Base.transaction do
    begin
      importer.each(&:save!)
    rescue
      raise ActiveRecord::Rollback
    end
  end
end
```

From the console, it's easier to see what we're trying to import, and once we include `Enumerable` in each importer class, we've got a lot more flexibility with regard to how we can debug failing imports:

```ruby
importer = SomeImporter.new(record)
first_invalid_record = importer.find { |r| !r.valid? }
```

Using `Enumerable#to_a`, we can refactor the migration to only open a transaction if there are records to import.

```ruby
records_which_should_be_imported.each do |record|
  importer = SomeImporter.new(record)
  new_records = importer.to_a

  if new_records.any?
    ActiveRecord::Base.transaction do
      begin
        new_records.each(&:save!)
      rescue
        raise ActiveRecord::Rollback
      end
    end
  end
end
```

My (admittedly informal) benchmarks indicate that this change results in a 10% speedup. It seems reasonable to attribute it to performing fewer queries inside of each transaction, and to nearly eliminating empty transactions altogether.

* * *

`Enumerable` is one of my favorite features of the Ruby core library, and I'm pleased with how it lends itself here to make a messy task like data migration relatively clean.

 [1]: https://github.com/gofullstack/chef-legacy
 [2]: http://www.exampler.com/ease-and-joy.html
 [3]: http://c2.com/cgi/wiki?MethodObject
