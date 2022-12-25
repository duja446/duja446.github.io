---
title: "Clojure - Futures, Delays and Promises"
date: 2022-12-25T12:52:49+01:00
tags: ["clojure"]
keywords: ["futures", "delays", "promises"]
---

# Futures, Delays and Promises

Futures, delays and promises are easy, lightweight tools for concurrent programming. 
They allow us to separate task definition, task execution and requiring the result.

## Futures

Futures are used to define a task and place it on another thread without requiring 
the result immediately.
```clojure
(future (Thread/sleep 4000)
        (println "prints after 4 seconds"))
(println "prints immediately")
```

When we want to get the result of a future task we can use the value that future macro 
returns. That value is a reference value that we can use to request the result. We request 
the result by *dereferencing* the future with eiter `deref` or `@`. Values of a future are 
cached and dereferencing a future will block if the future hasn't finished running.
```clojure
(let [result (future (println "this prints once")
                     (+ 1 1))]
  (println "deref: " (deref result))
  (println "@: " @result))
; => this prints once
; => deref: 2
; => @: 2
```

We can pass additional arguments to `deref` to tell it a number of milliseconds to wait 
along with the value to return if the call times out.
```clojure
(deref (future (Thread/sleep 1000) 0) 10 5)
; => 5
```

`realized?` is used to chech if the future is done running. 

## Delays

Delays are used to define a task without having to execute it or require the result 
immediately. They don't evaluate until we request it by using `force`. Also the result
of a delay is cached like the result of future.
```clojure
(def hello-delay
  (delay (println "Hello")
          "Hello world"))

(force hello-delay)
; => Hello
; => Hello world

(force hello-delay)
; => Hello world
```

## Promises

A promise is a reference to a value that may not yet be realized, or computed. A promise is 
created using the `promise` function, and its value can be set using `deliver`. 
```clojure
(def p (promise))

(defn compute-x []
  (Thread/sleep 1000)
  (deliver p 42))

(compute-x)

; block until p is delivered 
(println @p)
```
