---
title: "A simple reverse proxy in Go using Gin"
description: "Writing a reverse proxy in Go using Gin"
date:   2020-06-23T13:00:00+02:00
author: Sebastien Le Gall
categories: [Go]
---

Reverse proxies are great to handle logging, tracing, and authentication on applications you don't own or which you cannot modify the source code to make them do what you need.

A reverse proxy is nothing more than an HTTP server that handles a request and makes the request to a backend server. When doing so, it may add some headers, log data about the request or stop the request if authentication fails.

Go provide [a reverse proxy feature in its standard library](https://golang.org/src/net/http/httputil/reverseproxy.go). It's a great start, but you will probably find out it lacks some helper features such as logging or handling routing easily.

![gin](/img/article/go/gin.png)

On the other hand, [Gin](https://github.com/gin-gonic/gin/) is a very popular and great web-based application framework. It helps to build powerful REST API quickly, it has a lot of useful features and it's very fast since it uses [httprouter](https://github.com/julienschmidt/httprouter) under the hood. The idea is to handle HTTP requests with Gin and then proxy the request to the backend using the built-in reverse proxy.

The `ReverseProxy` struct contains a `Director` field. A director a function where you tell the reverse proxy what to do with the incoming request. It is where you may change the host, maybe add some headers, or even check the authentication.

```golang
remote, err := url.Parse("http://myremotedomain.com")
if err != nil {
	panic(err)
}

proxy := httputil.NewSingleHostReverseProxy(remote)
proxy.Director = func(req *http.Request) {
	req.Host = remote.Host
	req.URL.Scheme = remote.Scheme
	req.URL.Host = remote.Host
}
```

<!--more-->


## Writing a reverse proxy with Gin

The Gin engine provides a router to handle incoming requests. All you need to do is to declare a *catchall* route and handle all requests through the reverse proxy.

```golang
package main

import (
	"net/http"
	"net/http/httputil"
	"net/url"

	"github.com/gin-gonic/gin"
)

func proxy(c *gin.Context) {
	remote, err := url.Parse("http://myremotedomain.com")
	if err != nil {
		panic(err)
	}

	proxy := httputil.NewSingleHostReverseProxy(remote)
	//Define the director func
	//This is a good place to log, for example
	proxy.Director = func(req *http.Request) {
		req.Header = c.Request.Header
		req.Host = remote.Host
		req.URL.Scheme = remote.Scheme
		req.URL.Host = remote.Host
		req.URL.Path = c.Param("proxyPath")
	}

	proxy.ServeHTTP(c.Writer, c.Request)
}

func main() {

	r := gin.Default()

	//Create a catchall route
	r.Any("/*proxyPath", proxy)

	r.Run(":8080")
}

```

## Logging requests body

Most of the time, it's easy to know if the server have failed to respond by checking the returned status code in the log. But it doesn't help to understand *why* it failed.

One of the common solutions used to debug server applications is to log the request body.

But it's not always easy. Maybe you need to import a log package or class in a place where doing it implies a lot of changes in your code. Maybe you just cannot modify the source code directly.

With this simple reverse proxy, it's very easy to achieve this goal without even change the server application code. All you need to do is to log the request body in the `Director` func.

```golang
proxy.Director = func(req *http.Request) {
	b, _ := ioutil.ReadAll(req.Body)
	fmt.Println(string(b))
	//...
}
```