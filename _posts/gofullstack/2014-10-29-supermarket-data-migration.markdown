---
layout: post
title: "Supermarket Data Migration"
seo:
  title: "Supermarket Data Migration"
permalink: chef-supermarket-data-migration
date: 2014-10-29 14:57
category: elsewhere
author: Brian Cobb
original: http://gofullstack.com/chef-supermarket-data-migration
excerpt: >
  The trials and triumphs of migrating data from the old Chef community site to
  the Chef Supermarket.
tags:
  - Projects
  - Programming
  - Ruby
  - Supermarket
---

Supermarket was a fresh start for the Chef community site: a fresh framework, a fresh codebase, and a fresh perspective as to what role the site played for its community.
In this spirit, we built the majority of the existing community site's functionality more or less assuming that the ideal data model for a _fresh_ Supermarket would best suit the accumulated data of the community site.
This turned out to be not wholly naive!
That said, there were some interesting challenges behind getting all of the data from the old community site migrated over to Supermarket.

## In which we devise a migration process

From early on, we had planned on running Supermarket and community.opscode.com in parallel for a "soft-launch" period.
Leading up to and during the soft-launch period, there would some process by which new data on the community site would make its way to Supermarket.
Once the soft-launch was deemed complete, someone at Chef would flip the switch to redirect community.opscode.com to supermarket.getchef.com, and shortly thereafter the ongoing migration would cease.

From these requirements we built the [chef-legacy](https://github.com/gofullstack/chef-legacy) gem, which Supermarket bundled and used from a Sidetiq worker which ran every three hours.
From the worker, chef-legacy connected to both the community site's MySQL database and Supermarket's PostgreSQL database, found records in the former that did not yet exist in the latter, and used Supermarket's ActiveRecord models to migrate those records.
As opposed migrating the data directly between the two databases, this approach ensured that we weren't migrating data that Supermarket would consider invalid.
As it so happens, there was quite a bit of data that Supermarket considered invalid!

## In which a unique constraint saves the day

The most common type of invalid record was the duplicate `CookbookVersion`.

The heart of Supermarket's data model is the `Cookbook`, which is primarily a facade for every published version of the cookbook; those versions are represented as `CookbookVersion`s.
It stands to reason that there cannot be two `CookbookVersion` records associated with a given cookbook that have the same version number.
However, the old community site only had an ActiveRecord validation to enforce this constraint.
As the Rails documenation stipulates:

> This helper validates that the attribute's value is unique right before the object gets saved. It does not create a uniqueness constraint in the database, so it may happen that two different database connections create two records with the same value for a column that you intend to be unique.

Fortunately, Supermarket's database _does_ have a unique constraint to prevent duplicate pairs of `cookbook_id` and `version` values in the `cookbook_versions` table.

## In which the lack of a unique constraint creates more work

Unforunately, it's easy to not notice an opportunity to add a unique constraint and end up with a bunch of duplicated records.

One feature of Supermarket that wasn't present in the old community site is its ability to extract the list of dependencies from an artifact's `metadata.json` file.
Logically, the dependencies for a given version are unique: it doesn't make sense to depend on, say, the "apt" cookbook twice.
But because there was no constraint to prevent duplicate dependencies, an innocuous operator error (running two migration processes simultaneously) led to [duplicate sets of dependencies](https://github.com/opscode/supermarket/issues/442) on a number of cookbooks.

While [the fix](https://github.com/opscode/supermarket/pull/461) was not complicated, it would have been far more expedient to have the constraint in the first place.
If there's a lesson to be learned, it is to not be afraid to lean on database constraints to enforce domain truths.

## In which not every cookbook is a GZipped tarball

Knife, stove, and other cookbook publishing tools all package cookbooks as GZipped tarballs, and Supermarket's initial migration script assumed that all existing cookbook artifacts would be packaged the same way.
However, the old community site supported uploading any sort of artifact understood by libarchive, and _also_ allowed maintainers to upload artifacts via the web.
While testing chef-legacy, it became clear that maintainers had taken advantage of this feature to upload a wide variety of artifacts, and that `*.tgz`-only processing would not suffice to complete the migration.

To solve this, we temporarily introduced [backwards-compatible artifact processing](https://github.com/opscode/supermarket/pull/485).
And, if I may say, the implementation was quite nice: when it came time to reinstate `*.tgz`-only processing, all we needed to do was revert a [single commit](https://github.com/opscode/supermarket/commit/21114b276a68dd677038479f1cfab56b24778235)

## In which every cookbook must be migrated again

The bug with the most far-reaching consequences crept into existence via a seemingly-harmless data sanitization precuation.
Specifically, when importing dependency data, chef-legacy verified that the dependency constraint string was valid by instantiating a `Chef::VersionConstraint`.
It would then migrate that object's string representation, sort of like this:

```ruby
safe_constraint = Chef::VersionConstraint.new(unsafe_constraint)
cookbook_dependency.version_constraint = safe_constraint.to_s
cookbook_dependency.save!
```

Unfortunately, there was a [bug in `Chef::VersionConstraint`](https://github.com/opscode/chef/pull/1638) which led to constraints without patchlevels (e.g. `~> 0.1`) being migrated with a patchlevel of 0 (e.g. `~> 0.1.0`).
This, in turn, [caused problems for Berkshelf users](https://github.com/opscode/supermarket/issues/579) and ultimately required us to [re-process every migrated Cookbook](https://github.com/opscode/supermarket/pull/580).

## Closing Thoughts

I'd be remiss to not mention that a critical factor in the success of chef-legacy (aforementioned bugs notwithstanding) was that we had access to a snapshot of the community site's database.
This made it possible to build chef-legacy incrementally, and to test it locally.
Thanks to the snapshot, we squashed many, many bugs before they ever saw the light of production.

I think we started working on the migration at just the right time: we had all the pieces in our greenfield data model to house old data, and could use them to determine whether or not the model would support all of the existing data.
Migrating data from a legacy system to a new system is a unique opportunity to see how well greenfield ideals and assumptions comport with history, and to use the discrepancies to learn more about your users.
