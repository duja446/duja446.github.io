---
title: "Clojure - Pure Functions Powers"
date: 2022-12-18T19:21:02+01:00
tags: ["clojure"]
---

# Pure Functions Powers

Clojure provides functions that can derive new functions from old functions.

## comp

Composing pure functions is a common task in FP so Clojure provides `comp` to compose any 
number of functions to create a new function.
```clojure
((comp inc *) 2 3)
; => 7
```

Code written with `comp` is more elegant because it conveys more meaning. 

## memoize

Memoization takes advantage of referential transparency by storing the arguments
passed to a function and the return value of the function. With this subsequent calls 
to the function with the same arguments can return the result immediately.
```clojure
(defn timed-identity
  [x]
  (Thread/sleep 1000)
  x)

(timed-identity "hello")
; => "hello" after 1 second

(def memo-timed-identity (memoize timed-identity))
(memo-timed-identity "hello")
; => "hello" after 1 second

(memo-timed-identity "hello")
; => "hello" immediately
```

