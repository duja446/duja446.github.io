---
title: "Clojure - Writing Macros"
date: 2022-12-23T17:33:03+01:00
tags: ["clojure"]
keywords: ["macro", "evaluation model"]
---

# Writring Macros

## Macro structure

Macros are structured the same way as functions, they have a docstring, arguments and body. 
We can also use argument destructuring and can create multiple arity macros.

```clojure
(defmacro infix
  "Use infix syntax for calling functions"
  [[operand1 op operand2]]
  (list op operand1 operand2))
```

## Building Macros

### Distinguishing Symbols and Values

Macro body always tries to get the value of the symbol. We sometimes want to return the symbol 
itself without getting its value. To turn off this evaluation Clojure uses a single quote character 
`'`. 

```clojure
(defmacro my-print-error
  "Prints and returns a result of expression but errors"
  [expression]
  (list let [result expression]
        (list println result)
        (result)))

(defmacro my-print
  [expression]
  (list 'let ['result expression]
        (list 'println 'result)
        ('result)))
```

In this example we use quoting to stop Clojure from evaluating `let` and `println`. We also use 
it to bind `result` so we can get its value.

### Syntax Quoting

Syntax quoting is similar to regular quoting in that it returns unevaluated data structures, but 
they differ in that syntax quoting will return *fully qualified* symbols (symbols with the symbol's 
namespace included).
```clojure
'+
; => +

`
; => clojure.core/+
```

Another difference is that syntax quoting allows us to *unqoute* forms using tilde `~`.
```clojure
`(+ 1 ~(inc 1))
; => (clojure.core/+ 1 2)

`(+ 1 (inc 1))
; => (clojure.core/+ 1 (clojure.core/inc 1))
```

Syntax quoting leads to more readable and concise code.
