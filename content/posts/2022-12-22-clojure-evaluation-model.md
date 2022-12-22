---
title: "Clojure - Evaluation Model"
date: 2022-12-22T18:56:56+01:00
tags: ["clojure"]
keywords: ["evalution model", "homoiconic"]
---

# Evalution model

Clojure's evaluation model differs from the more 'standard' programming languages in that
it has two phases: read phase and evalution phase. For example to process `(+ 1 2)` Clojure 
first reads this text and converts it to a list data strucutre. This is then passed to the 
evaluator which looks up the function correspondind to `+` and applies it to 1 and 2.

Languages that process code in this way are called *homoiconic*. They allow us to reason 
about the code as a set of data structures that are manipulated programmatically.

## Compiled languages vs Clojure

Programming languages that are compiled or interpreted need to convert our code as text to 
machine instructions. They do that by converting our code into an *abstract syntax tree* (AST) 
which is a data structure that represents our program. In many programming languages AST is 
inaccessible within the programming language.

In Clojure this AST is a tree structured using Clojure lists, whose nodes are values. This 
allows us to change the AST in our program. This also means the evaluator doesn't care where 
its inputs comes from, it can come from the reader of we can supply it directly using `eval`.
```clojure
(def addition-list (list + 1 2))
(eval addition-list)
; => 3
```

At the end, this means we can manipulate the code after it has been read by the reader allowing 
us to do many interesting things. Code that does these manipulations is called `macro`.


