---
layout: post
title: "Jekyll and Vagrant: The Good, The Bad, and The Ugly"
seo:
  title: "Jekyll and Vagrant: The Good, The Bad, and The Ugly"
slug: jekyll-and-vagrant-the-good-the-bad-and-the-ugly
published: true
date: 2014-09-18 21:55
category: flapjacks
author: Brian Cobb
---

[gofullstack.com](http://gofullstack.com) is now running on Jekyll, and there's a Vagrantfile which automates a big chunk of local development setup.

For the last week, my workflow while writing blog posts and working on the site has been to run `vagrant up` inside the repo, and then to run `bin/jekyll serve -w --force_polling` in a tmux split. I'd then work in vim locally so that I didn't have to also set up the Vagrant box with vim and my dotfiles.

That I did not need to install anything but Vagrant + two Vagrant plugins locally to clone the repo and have a copy of the site running locally was fantastic. I'm making an effort to not pollute my host with development project cruft, and I succeeded in this one case. This is The Good.

This would be a near-perfect setup, but for the `--force_polling` flag passed to Jekyll. This means that, instead of reacting to filesystem events when files in the repo change, Jekyll must periodically sweep through every file in the project and look for changes in file modification times. Polling the project is by definition less efficient than reacting to filesystem events, and is in practice unbearably slow. This is The Ugly.

And so just now I've decided to compromise. It's a manual compromise for now; I just ran the following commands:

1.  `vagrant ssh -c 'sudo apt-get install -y vim git'`
1.  `vagrant ssh -c 'git clone git://github.com/bcobb/dotfiles .dotfiles'`
1.  `vagrant ssh -c 'cd .dotfiles && bin/dotme`
1.  `vagrant ssh -c 'vim +PluginInstall +qall`

Now both of my tmux splits are in some way running on the VM (one is running vim, the other is running jekyll), except now I've dropped the `--force_polling` flag and my rebuilding cycles are substantially faster. It's not an awful thing, I suppose, but it's not quite where I want to be. This is The Bad.

Eventually I'd like to figure out a better way to either work locally and not use polling, or work on the VM and not run three idiosyncratic commands to get my environment the way I want it. Some other day.
