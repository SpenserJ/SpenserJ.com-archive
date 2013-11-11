---
layout: post
title: "Dotfiles - A ~/ away from ~/"
date: 2013-11-11 11:45
comments: true
categories: 
- Server Administration
- Dotfiles
---
I've built out a nice little environment for myself, that I bring to each server I'm regularly working on through the use of [Dotfiles](https://github.com/SpenserJ/dotfiles). At its core, Zsh provides an attractive and helpful prompt, while Tmux provides panes, windows, and session resumption. There are Vim and Irssi configs as well, since I make heavy use of both.

The process of bringing an entire repository of config files to a new server, and then keeping all (1, 5, 10, 100) servers' configs in sync is a daunting task. Thankfully, there are tools that exist to handle the heavy lifting, and setting up your environment takes just a few minutes.

<!-- more -->

## Dependencies

The configuration I've built out relies on Zsh and Tmux, but you can choose to build an environment with something else, or just with a plain ol' bash prompt if you wish.

    $ sudo apt-get install git-core zsh tmux

## Homeshick

[Homesick](https://github.com/technicalpickles/homesick) is a Ruby Gem that simplifies the process of using Git Repositories for tracking your configuration files. The majority of servers that I work on have no need for Ruby, and I can't justify installing it just for my dotfiles, so I went looking for an alternative. Aanders Ingemann has developed a version written entirely in Bash, called [Homeshick](https://github.com/andsens/homeshick).

    $ git clone git://github.com/andsens/homeshick.git $HOME/.homesick/repos/homeshick

## Dotfiles

Now that we've installed Homeshick (yes, it really was that simple), we can go ahead and clone down a Dotfiles repository. At the end of the clone, Homeshick will ask if you want to symlink it, which will replace any existing configs with a link to our Dotfiles repository configs. Go ahead and say yes, and handle any conflicts as you deem appropriate.

    $ ~/.homesick/repos/homeshick/bin/homeshick clone SpenserJ/dotfiles

## Zsh + Tmux

Great! We have Homeshick (which I've aliased as Homesick), and we have the Dotfiles. If you were to reconnect right now, you wouldn't have full functionality though, as our `$SHELL` isn't set to Zsh yet. And since Zsh controls Tmux, you would have a very underwhelming experience with dotfiles. Change your shell, and then connect (in another window, just to be safe), and you should see a bar across the bottom, and a new prompt.

    $ chsh `whoami` -s `which zsh`

## Changing your dotfiles

You're probably going to make some changes to your dotfiles over the next few months, and you'll want to keep those changes backed up and synchronized across all of your servers. Homesick/Homeshick make this extremely easy to do so.

    # Track a new file
    $ homeshick track dotfiles ~/.vimrc
    
    # Commit your changes
    $ cd ~/.homesick/repos/dotfiles
    $ git add home/.vimrc
    $ git commit -m 'Using Solarized Light'
    $ git push
    
    # Updating your dotfiles on another server
    $ homeshick pull dotfiles
