---
title: "Clojure - Mistakes When Writing Macros"
date: 2022-12-24T12:48:45+01:00
tags: ["clojure"]
keywords: ["macros", "evaluation model"]
---

# Mistakes When Writing Macros

## Variable capture

This problem occurs when a variable name from within the macro clashes from the one outside.
For example if we had a macro that binds someting to `message` and we call that same macro 
with the variable also named `message`, only the `message` defined in the macro would be 
used.
```clojure
(def message "global message")
(defmacro example
  [& to-execute]
  (concat (list 'let ['message "macro message"])
          to-execute))

(example (println "The message is: " message))
; => The message is: macro message
```

To combat this we can use `gensym` to create a unique symbol. We can also use *auto-gensym* 
to shorten the syntax.
```clojure
(gensym)
; => G__456

(gensym 'message)
; => message3760

; auto-gensym
`(message#)
; => (message__4498__auto__)
```

Here is a fixed version of the code above
```clojure
(def message "global message")
(defmacro fixed-example
  [& to-execute]
  `(let [message# "macro message"]
    ~@to-execute
    (println "Message from the macro is: " message#)))

(fixed-example (println "The message is: " message))
; => The message is: global message
; => Message from the macro is: macro message
```

## Double Evaluation

Double evaluation occurs when a form passed to a macro as an argument gets evaluated more than once.
Here is an example:
```clojure
(defmacro report
  [to-try]
  `(if ~to-try
      (println (quote ~to-try) "was successful: " ~to-try)
      (println (qoute ~to-try) "was not successful: " ~to-try)))
```
Here the `to-try` would be evaluated twice: once for the `if`  and once in the `println`. This is 
a problem when `to-try` is some expensive code that takes time to run. With this macro we would 
double that execution time.

To fix this we can place `to-try` in a `let` expression to make it evaluate only once.
```clojure
(defmacro report
  [to-try]
  `(let [result# ~to-try]
      (if result#
        (println (qoute ~to-try) "was successful: " result#)
        (println (qoute ~to-try) "was not successful: " result#))))
```

## Too Many Macros

A thing with macros is they only compose with each other so by using them we can miss out 
on other kinds of composition. For example we can't use `doseq` with a macro or use a macro 
with `map`. We would need to write more macros to do what we want to. This creates more code 
where it is not really needed.


