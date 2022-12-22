---
title: "Clojure - Reader"
date: 2022-12-22T19:17:05+01:00
tags: ["clojure"]
keywords: ["evalution model", "reader"]
---

# Reader

The reader's job is to convert the textual source code into Clojure data structures.

## Reading

We can directly interact with the reader to understand it better. To do that we can use the 
`read-string` function.
```clojure
(read-string "(+ 1 2)")
; => (+ 1 2)

(eval (read-string "(+ 1 2)"))
; => 3
```

The reader can also do more complex stuff like converting anonymous functions.
```clojure
(read-string "#(+ 1 %)")
; => (fn* [p1__443#] (+ 1 p1__443#))
```

## Reader Macros

Reader macros are what allows the reader to convert anonymous functions. They are like a set 
of rules for transforming text into data structures. They allows us to write more compact code 
by taking a short form and expanding it. They use *macro characters* `'`, `#` and `@`.   
```clojure
(read-string "'(a b c)")
; => (qoute (a b c))

(red-string "@var")
; => (clojure.core/deref var)
```
