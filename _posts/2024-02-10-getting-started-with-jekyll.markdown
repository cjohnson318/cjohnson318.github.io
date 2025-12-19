---
layout: post
title: 'Getting Started With Jekyll'
date: 2024-02-10 19:00:00 -0700
categories: tech
tags: jekyll
---

I could not get the official Jekyll [documentation](https://jekyllrb.com/docs/installation/macos/) to work on my Mac, but I found a solution through [this](https://www.youtube.com/watch?v=UKB9ylw0G4U) video, from [this](https://talk.jekyllrb.com/t/need-help-with-chruby-unknown-ruby-ruby-3-1-1/7255) comment thread.

{% highlight console %}
brew install ruby@3.3
{% endhighlight %}

Next, set this ruby as your default Ruby in your `.zshrc` file.

{% highlight console %}
export PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH"
{% endhighlight %}

And then I ran:

{% highlight console %}
gem install bundle jekyll webrick
{% endhighlight %}

Now update your `PATH` again for your gems.

{% highlight console %}
export PATH="$HOME/.gem/ruby/3.3.0/bin:$PATH"
{% endhighlight %}

In the [documentation](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll) about creating a Jekyll site on GitHub Pages, step 7 should be:

{% highlight console %}
jekyll new --skip-bundle --force .
{% endhighlight %}

To run this locally, use the following.

{% highlight console %}
bundle exec jekyll serve --livereload
{% endhighlight %}
