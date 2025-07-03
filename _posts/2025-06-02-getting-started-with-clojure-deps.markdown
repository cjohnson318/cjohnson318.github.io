---
layout: post
title: "Getting Started with Clojure Deps"
date: 2025-06-02 00:00:00 -0700
tags: clojure
---

I've been wanting to really get into using Clojure with the newer and preferred
EDN tooling, over the older Leinengen tooling.


## Initial Setup

First we'll set up our directory and write a Hello World application.

{% highlight console %}
brew install clojure 
mkdir learn-deps
cd learn-deps
touch deps.edn
{% endhighlight %}

then in `deps.edn`,

{% highlight clojure %}
{
  :paths ["src/clj"]
  :deps {org.clojure/clojure {:mvn/version "1.12.1"}
}
{% endhighlight %}

{% highlight console %}
mkdir -p src/clj/learn_deps
cd src/clj/learn_deps
touch core.clj
{% endhighlight %}

{% highlight clojure%}
(ns learn-deps.core)

(defn -main []
  (println "Hello World!")
)
{% endhighlight %}

Then run from the root `learn-deps/` directory as,

{% highlight console %}
clj -M -m lean-deps.core
{% endhighlight %}


## Using Aliases

We can simplify this command by adding an alias to `deps.den`,

{% highlight clojure %}
{
  :paths ["src/clj"]
  :deps {org.clojure/clojure {:mvn/version "1.12.1"}
  :aliases {:run {:main-opts ["-m" "learn-deps.core"]}}
}
{% endhighlight %}

Then we can run this with the `:run` alias as,

{% highlight console %}
clj -M:run
{% endhighlight %}


## Using Calva from VS Code

As much as I love vim, I switched to VS Code for it's Calva integration.

Start an interactive REPL with Calva in VS Code by adding this to `deps.edn`,

{% highlight clojure %}
{
  :paths ["src/clj"]
  :version {org.clojure/clojure {:mvn/version "1.12.1"}}
  :aliases {
    :run {:main-opts ["-m" "learn-deps.core"]}
    :repl {:extra-deps {nrepl/nrepl {:mvn/version "1.3.1"}}}       
  }
} 
{% endhighlight %}

Then in VS Code, with Calva installed, type `>jack` into the command palette,
and select, `Calva: Start a PRoject REPL and Connect...`.

Calva should start up a REPL, then we can look for `>load` in the command palette,
and select, `Calva: Load/Evaluate Current File...`

Then in the REPL window you can run `(-main)` and see the output of your main function.

