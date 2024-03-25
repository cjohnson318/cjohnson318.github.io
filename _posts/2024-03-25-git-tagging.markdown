---
layout: post
title:  "Tagging in Git"
date:   2024-03-25 00:00:00 -0700
categories: git
---

Tags are used to track versions in Git. There are lightweight tags and 
annotated tags, you should generally use the annotated tags. These tags are
created using the following command.

{% highlight console %}
git tag -a v1.0 -m 'Finally made it to v1.0'
{% endhighlight %}

Show more information about a tag or version using the `show` command. 

{% highlight console %}
git show v1.0
{% endhighlight %}

List all tags using the `tag`. Wierdly, `-l` and `--list` are optional. 

{% highlight console %}
git tag
git tag -l 
git tag --list
{% endhighlight %}

Read the documentation [here](https://git-scm.com/book/en/v2/Git-Basics-Tagging)
