[![GoDoc](https://godoc.org/github.com/didip/tollbooth?status.svg)](http://godoc.org/github.com/didip/tollbooth)
[![license](http://img.shields.io/badge/license-MIT-red.svg?style=flat)](https://raw.githubusercontent.com/didip/tollbooth/master/LICENSE)

## Tollbooth

This is a generic middleware to rate-limit HTTP requests.

**NOTE 1:** This library is considered finished.

**NOTE 2:** In the coming weeks, I will be removing thirdparty modules and moving them to their own dedicated repos.


## Five Minutes Tutorial
```go
package main

import (
    "github.com/didip/tollbooth"
    "net/http"
    "time"
)

func HelloHandler(w http.ResponseWriter, req *http.Request) {
    w.Write([]byte("Hello, World!"))
}

func main() {
    // Create a request limiter per handler.
    http.Handle("/", tollbooth.LimitFuncHandler(tollbooth.NewLimiter(1, time.Second), HelloHandler))
    http.ListenAndServe(":12345", nil)
}
```

## Features

1. Rate-limit by request's remote IP, path, methods, custom headers, & basic auth usernames.
    ```go
    limiter := tollbooth.NewLimiter(1, time.Second)

    // or create a limiter with expirable token buckets
    // This setting means:
    // create a 1 request/second limiter and
    // every token bucket in it will expire 1 hour after it was initially set.
    limiter = tollbooth.NewLimiterExpiringBuckets(1, time.Second, time.Hour, 0)

    // Configure list of places to look for IP address.
    // By default it's: "RemoteAddr", "X-Forwarded-For", "X-Real-IP"
    // If your application is behind a proxy, set "X-Forwarded-For" first.
    limiter.SetIPLookups([]string{"RemoteAddr", "X-Forwarded-For", "X-Real-IP"})
 
    // Limit only GET and POST requests.
    limiter.SetMethods([]string{"GET", "POST"})

    // Limit based on basic auth usernames.
    // You can first define them on-load.
    limiter.SetBasicAuthUsers([]string{"bob", "jane", "didip", "vip"})
    // But you can also add them later.
    limiter.AddBasicAuthUsers([]string{"sansa"})
    // As well removing them later.
    limiter.RemoveBasicAuthUsers([]string{"vip"})


    // Limit request headers containing certain values.
    // Typically, you prefetched these values from the database.
    limiter.Headers = make(map[string][]string)
    limiter.Headers["X-Access-Token"] = []string{"abc123", "xyz098"}
    ```

2. Each request handler can be rate-limited individually.

3. Compose your own middleware by using `LimitByKeys()`.

4. Customize your own message or function when limit is reached.

    ```go
    limiter := tollbooth.NewLimiter(1, time.Second)

    // Set a custom message.
    limiter.SetMessage("You have reached maximum request limit.")

    // Set a custom content-type.
    limiter.SetMessageContentType("text/plain; charset=utf-8")

    // Set a custom function for rejection.
    limiter.SetRejectFunc(func() { fmt.Println("A request was rejected") })
    ```

5. Tollbooth does not require external storage since it uses an algorithm called [Token Bucket](http://en.wikipedia.org/wiki/Token_bucket) [(Go library: golang.org/x/time/rate)](//godoc.org/golang.org/x/time/rate).


# Other Web Frameworks

Sometimes, other frameworks require a little bit of shim to use Tollbooth. These shims below are contributed by the community, so I make no promises on how well they work. The one I am familiar with are: Chi, Gin, and Negroni.

* [Chi](https://github.com/didip/tollbooth_chi)

* [Echo](https://github.com/didip/tollbooth_echo)

* [FastHTTP](https://github.com/didip/tollbooth_fasthttp)

* [Gin](https://github.com/didip/tollbooth_gin)

* [GoRestful](https://github.com/didip/tollbooth_gorestful)

* [HTTPRouter](https://github.com/didip/tollbooth_httprouter)

* [Iris](https://github.com/didip/tollbooth_iris)

* [Negroni](https://github.com/didip/tollbooth_negroni)
