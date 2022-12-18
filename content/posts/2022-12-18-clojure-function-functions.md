---
title: "Clojure - Function Functions"
date: 2022-12-18T09:57:47+01:00
draft: true
---

# Function Functions

## apply

`apply` expands a seqable data structure so it can be passed to a function that expects 
a rest parameter.
```clojure
(max 0 1 2)
; => 2

(max [0 1 2])
; => [0 1 2]

(apply max [0 1 2])
; => 2
```

## partial

`partial` takes a function and any number of arguments. It then returns a function which, when called, 
calls the original function with the supplied arguments and the new ones.
```clojure
(def add10 (partial + 10))
(add10 3)
; => 13

(add10 10)
; => 20
```

## complement

`complement` is a simple function that just gives the negation of a Boolean function. 
```clojure
(def my-pos? (complement neg?))
(my-pos? 1)
; => true

(my-pos? -1)
; => false
```

