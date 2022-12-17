---
title: "Clojure - Functions"
date: 2022-12-17T08:52:40+01:00
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

### Parameters and Arity

Clojure functions support arity overloading meaning one function can be defined to do different things
depending on the number of arguments passed to it.
```clojure
(defn multi-arity
  ([first-arg second-arg]
    (do-thing first-arg second-arg))
  ([first-arg]
    (do-thing first-arg)))
```

Clojure allows defining of variable-arity functions by including a *rest parameter*(&)
```clojure
(defn greet
  [name]
  (str "Hello, " name " !"))

(defn greet-many
  [& people]
  (map greet people))

(greet-many "Bill" "Anne")
; => ("Hello, Bill !" "Hello, Anne !")
```

### Destructuring

Destructuring is used to concisely bind names to values within a collection.
```clojure
(defn first-thing
  [[f]]
  f)

(first-thing [1 2 3 4])
; => 1
```

Maps can also be destructured in a similar way. 
```clojure
(defn greet-full
  [{name :name surname :surname}]
  (str "Hello, " name " " surname " !"))

(greet-full {:name "Bill" :surname "Clinton"})
; => "Hello Bill Clinton !"

; Shorter syntax
(defn greet-full
  [{:keys [name surname]}]
  (str "Hello, " name " " surname " !"))
```

## Anonymous Functions

Functions that don't have names are called anonymous functions. In Clojure we can define 
them in two ways:
```clojure
(fn [x] (* x 3))
```
```clojure
#(* % 3)
```

The percent sign indicated the argument passed to the function. When a anonymous function takes
multiple parameters we can distinguish them like this: %1, %2, %3 ...

## Returining Functions

Functions can return other functions and the returned functions are *closures* meaning they can access
all the variables that were in scope when the function was created.
```clojure
(defn inc-maker
  [inc-by]
  #(+ % inc_by))

(def inc3 (inc-maker 3))

(inc3 7)
; => 10
```
