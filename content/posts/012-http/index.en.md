+++
title = 'HTTP'
date = 2020-05-02T09:00:00+03:00
draft = false
tags = ['http', 'client', 'server', 'middleware']
url = '/en/post/http.html'

[quiz]
  [[quiz.questions]]
    question = "What interface must be implemented to create an HTTP handler in Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "`http.Handler` with `ServeHTTP` method"
      correct = true
    [[quiz.questions.answers]]
      text = "http.Server interface"
      correct = false
    [[quiz.questions.answers]]
      text = "`http.Request` interface"
      correct = false
  
  [[quiz.questions]]
    question = "What does the standard Go router support?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Plain URL comparison only"
      correct = true
    [[quiz.questions.answers]]
      text = "Pattern matching with parameters"
      correct = false
    [[quiz.questions.answers]]
      text = "Regular expressions"
      correct = false
  
  [[quiz.questions]]
    question = "What is middleware used for?"
    type = "multiple-choice"
    [[quiz.questions.answers]]
      text = "Logging"
      correct = true
    [[quiz.questions.answers]]
      text = "Statistics gathering"
      correct = true
    [[quiz.questions.answers]]
      text = "Authorization checking"
      correct = true
    [[quiz.questions.answers]]
      text = "Database queries"
      correct = false
  
  [[quiz.questions]]
    question = "How is middleware typically implemented in Go?"
    type = "single-choice"
    [[quiz.questions.answers]]
      text = "Using nested handlers"
      correct = true
    [[quiz.questions.answers]]
      text = "Using special middleware package"
      correct = false
    [[quiz.questions.answers]]
      text = "Using annotations"
      correct = false
+++

All examples from the article are located in the github repository.

Let's take a look at HTTP tools in GO.

<!--more-->

Among other standard GO packages, there is the `net/http` package, which implements both HTTP client and server functionality.

## HTTP Server

The simplest way to start an HTTP server is to call `http.ListenAndServe`:

```go
package main

import (
	"log"
	"net/http"
)

type Handler struct {}

func (h *Handler) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	_, err := w.Write([]byte("hello"))
	if err != nil {
		log.Println(err)
	}
}

func main() {
	log.Println(http.ListenAndServe("127.0.0.1:9090", &Handler{}))
}
```

We just ran a server on port 9090.

The second parameter of `http.ListenAndServe` is an `http.Handler` interface implementer. The interface requires a `ServeHTTP(w http.ResponseWriter, req *http.Request)` function. This function will process all requests on our server:

```bash
curl localhost:9090/someurl
hello

curl localhost:9090/secondurl
hello
```

We can also configure the server to process a specific URL, for example `/hello`:

```go
package main

import (
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/hello", func(w http.ResponseWriter, req *http.Request) {
		_, err := w.Write([]byte("hello"))
		if err != nil {
			log.Println(err)
		}
	})

	log.Println(http.ListenAndServe(":9090", nil))
}
```

How does it work? When we call `http.HandleFunc`, the path and handler function are added to the `net/http.DefaultServeMux` struct, which holds all of them. When we call a particular URL, `net/http.DefaultServeMux` is used to find the particular handler. If no handler is found, a standard 404 response is generated.

The standard router implements only plain URL comparison to each of the registered URLs. Pattern matching, such as matching an expression like `/hello/{page}`, where the `page` parameter can be different between several calls, is impossible with the standard router. To implement that, we need either to implement pattern matching ourselves or to use one of the router libraries.

## Custom Router

We could use any other router, for example `gorilla/mux`. It supports pattern matching.

```go
func main() {
	r := mux.NewRouter()
	r.HandleFunc("/products/{key}", ProductHandler)
	r.HandleFunc("/articles/{category}/", ArticlesCategoryHandler)
	r.HandleFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler)
	http.Handle("/", r)
}
```

Next, in the handler we can grab the variables from the URL:

```go
func ArticlesCategoryHandler(w http.ResponseWriter, r *http.Request) {
	vars := mux.Vars(r)
	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, "Category: %v\n", vars["category"])
}
```

## Middleware

In a standard web service, there is not only handler code that runs on every request. There is also some common code that performs general work — logging, statistics gathering, authorization checking, etc. This common code is called middleware.

There is no explicit implementation of middlewares in the GO standard library. But anyone can implement something like that by using nested handlers. Let's take a look at a logging middleware that prints every requested URL to the console:

```go
package main

import (
	"log"
	"net/http"
)

func loggingMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		log.Printf("%s requested", r.URL.Path)
		next.ServeHTTP(w, r)
	})
}

func main() {
	handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("hello"))
	})

	http.Handle("/", loggingMiddleware(handler))
	log.Println(http.ListenAndServe(":9090", nil))
}
```

On every request to our server, we now can see the following data in the console:

```
2020/05/01 18:09:42 / requested
2020/05/01 18:09:58 /someurl requested
```

Libraries like `gorilla/mux`, `echo`, or `gin` can significantly help with middleware implementation. Also, the most popular tasks for middleware are already covered in many libraries.

## HTTP Client

In the `net/http` package, there is an `http.Client` struct that is responsible for HTTP client requests:

```go
client := http.Client{}

resp, err := client.Get("https://golangforall.com/en/")
if err != nil {
	log.Fatal(err)
}

buf, err := ioutil.ReadAll(resp.Body)
if err != nil {
	log.Fatal(err)
}

fmt.Println(string(buf))

// some html is printed
```

The standard HTTP client in GO looks simple, but it covers all possible tasks.

Some of the configuration is covered by the `Transport` field in the `Client` struct.
