---
title: "Clojure - Syntax"
date: 2022-12-16T17:43:56+01:00
keywords: ["clojure"]
---

# Syntax

Clojure syntax differs from many programming languages in that that it uses parantheses
and indentation to construct code. The reason for this is that Clojure is a Lisp, a variant of
the Lisp programming language.

## Forms 

Clojure recognizes two kinds of structures:
- literal representations of data structures (numbers, strings ...)
- operations

Term *form* refers to an expression in Clojure.
```clojure
1
"hello world"
["vector" "of" "string"]
```
Operations are how we do things. They are written as *opening paranthesis, operator, operand, closing parantheses*
```clojure
(operator operand1 operand2 ... operandn)
(+ 1 2)
(+ 1 2 6 7)
```

This style of writing operations is uniform across the whole language unlike a language 
like JavaScript which uses a mix of notations

## Control Flow

### **if**

```clojure
(if true
  "Displays if true"
  "Displays if false")
; => Displays if true

(if false
  "Displays if true")
; => nil
```

### **do**

The do operator is used to wrap multiple forms and run each of them.
```clojure
(if true
  (do (println "Success")
      "Displays if true")
  (do (println "Failure")
      "Displays if false"))
; => Success
; => Displays if true
```

### **when**

The when operator is a combination of `if` and `do` but with no `else` branch.
```clojure
(when true
  (println "Success")
  "boom!!")
; => Success
; => boom!!
```

### **boolean expressions**

Clojure provides all equality testing functions like `=` `or` `and`. 

## Naming values with def

`def` can be used to binf a name to a value

```clojure
(def a_vector
  [1 2 3 4])

vector
; => [1 2 3 4]
```

