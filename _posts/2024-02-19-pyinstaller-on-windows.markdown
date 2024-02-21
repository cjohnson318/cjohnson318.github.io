---
layout: post
title:  "PyInstaller on Windows"
date:   2024-02-19 19:00:00 -0700
categories: python
---

PyInstaller is amazing, but it doesn't do cross-compilation. Furthermore, things
that go well on your Mac might need more work on a Windows machine.

I struggled a little bit figuring out how to run PyInstaller on Windows after I
had installed it using `pip`. Then I had trouble figuring out how to tell it
about custom modules that I had written for my script that lived in a
subdirectory named `lib`.

Eventually this worked. Here `lib` is a subdirectory of the working directory,
and `script.py` is the script that I'm interested in turning into an EXE file.

{% highlight console %}
py -m PyInstaller -p .\lib .\script.py --onefile
{% endhighlight %}

