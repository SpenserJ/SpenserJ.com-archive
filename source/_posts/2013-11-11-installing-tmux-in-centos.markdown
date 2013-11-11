---
layout: post
title: "Installing tmux in CentOS"
date: 2013-11-11 11:53
comments: true
categories: 
---
I've recently started a job at [Simple Simple Advertising](http://simplesimple.ca/), and have spent my first week optimizing servers and configuring proper dev environments, all of which are CentOS. While CentOS is a rock-solid OS, I'm not particularly fond of the 100% binary compatibility concept, since I don't try to run 100% of the available software on a single server, and test my stacks before they reach production. I'm far more interested in having semi-recent packages with security fixes, and tools that make my life easier. Tools like my [Dotfiles](http://spenserj.com/blog/2013/11/11/dotfiles-a-home-away-from-home/) and the configurations that they contain, along with tmux and Zsh for managing multiple windows within a single ssh connection. Sadly, CentOS' version of Tmux does not work. More specifically, tmux won't start with the Zsh config I've built, so I built a more recent version from source.

<!-- more -->

## Dependencies

    $ sudo yum install gcc make automake pkgconfig wget git-core ncurses-devel
    $ mkdir ~/src

## LibEvent

Tmux relies on LibEvent, which I could also `yum install`, but the version has been pinned there as well. Since I'm running x64, we'll have to symlink it to the `/usr/lib64` folder after install.

    $ cd ~/src
    $ wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
    $ tar -zxvf libevent-2.0.21-stable.tar.gz
    $ cd libevent-2.0.21-stable
    $ ./configure
    $ make
    $ sudo make install
    $ sudo ln -s /usr/local/lib/libevent-2.0.so.5 /usr/lib64

## Tmux

Now that we're set up with dependencies for tmux, clone the repository and build it.

    $ cd ~/src
    $ git clone git://git.code.sf.net/p/tmux/tmux-code
    $ cd tmux-code
    $ sh autogen.sh
    $ ./configure
    $ make
    $ sudo make install

Now you have a recent version of tmux that works properly with Zsh+[Prezto](https://github.com/sorin-ionescu/prezto).  It's good to be `~/`
