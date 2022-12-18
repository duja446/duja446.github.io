---
title: "Clojure - Working With Immutable Data Structures"
date: 2022-12-18T19:01:09+01:00
tags: ["clojure"]
---

## Recursion Insted of for/while

I hate looping through a list in a language like JavaScript, because this creates imperative
code that has side effects. Recursive functions may be difficult to read at first but with
enough practice they become easier to read then imperative functions created with loops.
Here is an example of how to build a sum function recursivly
```clojure
(defn sum
  ([vals] (sum vals 0))
  ([vals accumulating-total]
    (if (?empty vals)
      accumulating-total
      (sum (rest vals) (+ (first vals) accumulating-total)))))
```

When writing recursive functions in Clojure `recur` should be used to provide optimization to the function.
```clojure
(defn sum
  ([vals] (sum vals 0))
  ([vals accumulating-total]
    (if (empty? vals)
      accumulating-total
      (recur (rest vals) (+ (first vals) accumulating-total)))))
```

This recuring won't in fact create thousands of intermediate values because Clojure's immutable
data structures are implemented using *structural sharing* which allows it to efficiently create 
new versions of data structures that share some of the same structure as the original data structure.

## Function Composition Instead of Attribute Mutation

Function composition is the practice of combining functions in such a way that the return value
of one function is passed as an argument to another. With this technique we can craft complex 
functions from simple ones. This also decouples data from functions making functions easier to 
use and adapt.

