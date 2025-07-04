---
layout: post
title: "Clojure Simple Web Server"
date: 2025-07-03 00:00:00 -0700
tags: clojure
---

After learning about setting up a Clojure project with `deps.edn`, I wanted to
build a simple web server. I found a video, linked below, and worked out a
simple example. Next, I'd like to build a ClojureScript frontend.


## Initial Setup ##

First we'll set up our directory and write a Hello World application.

{% highlight console %}
brew install clojure 
mkdir simple-web-server
cd simple-web-server
touch deps.edn
{% endhighlight %}


## deps.edn ##

Then we can write our dependencies file in `deps.edn`,

{% highlight clojure %}
{
  :paths ["src/clj"]
  :deps {
            org.clojure/clojure {:mvn/version "1.12.1"}
            http-kit/http-kit {:mvn/version "2.8.0"}
            compojure/compojure {:mvn/version "1.7.1"}
            cheshire/cheshire {:mvn/version "5.10.0"}}
  :aliases {
    :run {:main-opts ["-m" "simple-web-server.core"]}
    :repl {:extra-deps {nrepl/nrepl {:mvn/version "1.3.1"}}}       
  }
} 
{% endhighlight %}


## Simpler Server ##

Then we can write a lightweight server,

{% highlight console %}
mkdir -p src/clj/simple_web_server
cd src/clj/simple_web_server
touch core.clj
{% endhighlight %}

{% highlight clojure%}
(ns simple-web-server.core
  (:require
   [compojure.core :refer [defroutes GET]]
   [cheshire.core :as json]
   [org.httpkit.server :refer [run-server]]))

(defroutes app
  (GET "/html" [] "<h1>HELLO HTML ENDPOINT</h1>")
  (GET "/json" [] {:status 200
                   :headers {"Content-Type" "application/json"}
                   :body (json/generate-string {:message "Hello JSON endpoint"})})
  )

(defn -main []
  (run-server app {:port 4321})
)
{% endhighlight %}

Then run from the root `simple-web-server/` directory as,

{% highlight console %}
clj -M:run
{% endhighlight %}


## References ##

This is adapted and update from an older YouTube [video](https://www.youtube.com/watch?v=yVk_ImBQqms) by Kelvin Mai.
