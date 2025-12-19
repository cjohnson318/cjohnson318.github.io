---
layout: post
title: "Clojure Simple Web Server"
date: 2025-07-03 00:00:00 -0700
categories: tech
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

Here, the context function allows us to next routes. In the second `json` route,
we're using a keyword `:id` in the route, which is passed from the request, and
into the function, as an argument.

{% highlight clojure%}
(ns simple-web-server.core
  (:require
   [compojure.core :refer [defroutes context GET POST]]
   [compojure.route :as route]
   [cheshire.core :as json]
   [org.httpkit.server :refer [run-server]]))

(defroutes app
  (GET "/html" [] "<h1>HELLO HTML ENDPOINT</h1>")
  (context "/json" []
    (GET "/" [] {:status 200
                     :headers {"Content-Type" "application/json"}
                     :body (json/generate-string {:message "Hello JSON endpoint"})}) 
    (GET "/:id" [id] {:status 200
                 :headers {"Content-Type" "application/json"}
                 :body (json/encode {:message "Hello JSON endpoint" :id id})})
    (POST "/" request
      (let [body (json/parse-string (slurp (:body request)) true)]
        {:status 200
         :headers {"Content-Type" "application/json"}
         :body (json/encode {:message "Received JSON data" :data body})})))
    )

  (route/not-found {:status 404
                    :headers {"Content-Type" "application/json"}
                    :body (json/encode {:error "Not Found"})}))

(defn start-server []
  (println "Starting server on port 4321...")
  (run-server app {:port 4321}))

(defn -main []
  (start-server))

(defonce server (atom nil))

(defn start []
  (reset!  server (start-server)))

(defn stop []
  (when @server
    (@server :timeout 100)))

(defn restart []
  (stop)
  (start))
{% endhighlight %}

Then run from the root `simple-web-server/` directory as,

{% highlight console %}
clj -M:run
{% endhighlight %}


## Testing the Endpoints ##

And now we can test this by using httpie. Here is the `GET` request:

{% highlight console %}
$ http localhost:4321/json/1
HTTP/1.1 200 OK
Content-Type: application/json
Date: Sun, 6 Jul 2025 02:22:37 GMT
Server: http-kit
content-length: 42

{
    "id": "1",
    "message": "Hello JSON endpoint"
}
{% endhighlight %}

Here is a `POST` request:

{% highlight console %}
$ http POST localhost:4321/json/ foo=bar   
HTTP/1.1 200 OK
Content-Type: application/json
Date: Sun, 6 Jul 2025 02:34:32 GMT
Server: http-kit
content-length: 53

{
    "data": {
        "foo": "bar"
    },
    "message": "Received JSON data"
}
{% endhighlight %}

## References ##

This is adapted and update from an older YouTube [video](https://www.youtube.com/watch?v=yVk_ImBQqms) by Kelvin Mai.
