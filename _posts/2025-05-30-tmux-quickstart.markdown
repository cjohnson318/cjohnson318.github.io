---
layout: post
title: "tmux Quickstart"
date: 2025-05-30 00:00:00 -0700
tags: linux
---

There are a couple of great tmux resources out there, like 
[tmuxcheatsheet](https://tmuxcheatsheet.com), and a lot of resources
that are covered in AI generated ads for wrinkle cream. I think the best way to
present this information is to start with the very minimum, and then build up.

## Custom Config

This is my tmux config that I got from somewhere a couple of years ago. The
default way to use tmux commands is to use the key combination `CTRL-b`, but I 
find `CTRL-SPACE` more ergonomic on my keyboard, so I configured that here.

{% highlight console %}
set -g mouse on

set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible' 
set -g @plugin 'christoomey/vim-tmux-navigator'

set -g default-terminal "screen-256color"

unbind C-b
set -g prefix C-Space
bind C-Space send-prefix

unbind %
bind | split-window -h

unbind '"'
bind - split-window -v

# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
run '~/.tmux/plugins/tpm/tpm'
{% endhighlight %}

## Sessions

The main idea is that you have on-going sessions. Then each session can have
one or more windows, and each window can have one or more panes. Create a named
session like this,

{% highlight console %}
tmux new -s <session-name>
{% endhighlight %}

Detach from a session like this,

{% highlight console %}
CTRL-SPACE d
{% endhighlight %}

Then re-attach to a session like this,

{% highlight console %}
tmux a -t <session-name>
{% endhighlight %}

## Panes

Panes are created in Windows. Using the config above, you can split a window
into side-by-side panes as,

{% highlight console %}
CTRL-SPACE |
{% endhighlight %}

And into panes above and below as,

{% highlight console %}
CTRL-SPACE -
{% endhighlight %}

And then you can navigate between panes by using `CTRL-SPACE <arrow>`.

