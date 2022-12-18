---
title: "Clojure - Pure Functions"
date: 2022-12-18T11:04:34+01:00
tags: ["clojure"]
---

# Pure Functions

Pure functions are functions that:
1. always return the same result if given the same arguments - *referential transparency*
2. don't cause side effects

Pure functions are easier to reason about because they are isolated and are unable to 
impact other parts of the program. Basicly they are stable and problem free as arithmetic.

## Referential Transparency

To achieve referential transparency pure function rely only on their own argument and 
immutable values to determine their return value. Here is an example of a pure function 
and a 'dirty' functions.
```clojure
(defn greet
  [name]
  (str "Hello, " name))

(greet "Duja")
; => "Hello, Duja"

(defn maybe-greet
  [name]
  (if (> (rand) 0.5)
    "Hello, " name
    "..."))
```

## No Side Effects

A program has to have some side effects like writing to a disk, rendering to a screen and so 
on. But this sade effect can be potentially harmful because you can't always be sure how 
running a piece of code impacts other parts of the system. When calling a function without 
side effect you only need to worry about the relationship between the input and the output
of the function.
Functions with side effects are also harder to understand and test. Having to worry about
the whole system not just the one function is certinly hard.
Clojure makes the job of writing functions without side effect easy because all of its core
data structures are immutable meaning they can't be changed in place. Working with immutable
structures is quite different from working with their mutable counterparts so this style of
programming requires getting some used to.

