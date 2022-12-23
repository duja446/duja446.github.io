---
title: "Clojure - Evaluator"
date: 2022-12-23T16:17:47+01:00
tags: ["clojure"]
keywords: ["evaluation model", "evaluator"]
---

# Evaluator

Clojure's evaluator is like a function that takes a data structure as an argument, processes 
the data structure using rules corresponding to the data structure's type, and return a result.
Different data structures are evaluated differently.

## Things that evaluate to themselves

```clojure
true
; => true

false
; => false

{}
; => {}

:abc
; => :abc

()
; => ()
```

## Symbols

Clojure uses symbols to name things like functions and data and evalutes them by resolving them. 
Resolving a symbol means Clojure traverses any bindings created and then looks up the symbol's 
entry in a namespace mapping.

```clojure
(def x 10)
(+ x 3)
;=> 13

(read-string ("+"))
; => +

(type (read-string "+"))
; => clojure.lang.Symbol

```

On their own, symbols and their referents don't actually do anything, Clojure performs work by 
evaluating lists.

## Lists

If the list is not empty, the first element is evaluated as a call to the functions referenced 
by that symbol.

### Functions Calls

When performing a function call, each operand is fully evaluated and then passed to the 
function as an argument. For example if we have `(+ 1 (+ 2 3))` we first evaluated `1` to 
itself and the need to evaluate `(+ 2 3)`. This is then evaluated to `5` and then `1` and `5` 
are passed to `+` as operands. 

### Special Forms

Special forms differ in that each of their operands don't need to be fully evaluated. For example 
when calling `(if true 1 2)`, the evaluator only need to evaluate the first operand so it doesn't 
evalue `2`.

## Macros

Macros are a convenient way to manipulate lists before Clojure evaluates them. They are a lot like 
functions but they are executed between the reader and the evaluator allowing them to process structures 
from the reader before they get to the evaluator.

```clojure
(defmacro ignore-last-operand
  [function-call]
  (butlast function-call))

(ignore-last-operand (+ 1 2 200))
; => 3
```

Call to a function would first evaluate `(+ 1 2 200)` but all operands of a macro are not evaluated 
before they are used so insted of `203`, macro gets the full `(+ 1 2 200)`.  
