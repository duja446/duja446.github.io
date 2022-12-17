---
title: "Clojure - Functions"
date: 2022-12-17T08:52:40+01:00
draft: true
keywords: ["clojure"]
---

# Functions

Functions are a primary building block in Clojure. They can be used to craft complex
programs from small and simple blocks.

## Calling functions

```clojure
(+ 1 2 3 4)
(* 1 2 3 4)
(first [1 2 3 4])
```

In FP languages we treat functions as **first class citizens** meaning we can use them 
like any other datatype. This allows us to pass them to other functions making *higher order functions* - 
functions that take functions as parameters. An example of such function would be `map` which creates 
a new list by applying a function to each member of a collection.

```clojure
(map inc [0 1 2 3])
; => [1 2 3 4]
```

## Defining functions

Function definitions are composed of five parts:
- `defn` 
- function name
- an optional docstring
- parameters listed in brackets
- function body

### The Docstring

Docstring represents a useful way to describe and document code. Dostrings can be viewed in 
REPL with `(doc fn-name)`.

## Parameters and Arity
