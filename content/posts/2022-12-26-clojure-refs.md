---
title: "Clojure - Refs"
date: 2022-12-26T19:38:19+01:00
tags: ["clojure"]
keywords: ["refs", "commute", "alter"]
---

# Refs

Refs are use to manage mutable, shared state like atoms, but they allow us to express that 
an event sould update the state of more than one identity simultaneously. They allow us to 
update the state of multiple identities using transaction semantics. Transaction have three 
features: 
* They are **atomic**, meaning that all refs are updated or none of them are
* They are **consistent**, meaning that refs always have a valid state
* They are **isolated**, meaning that transaction behave as if they are executed serially

An example usage of refs would be if we wanted to transfer money from one bank account to 
another. We would need to make a transaction that takes money from one account and adds the 
same amount to the other. 
```clojure
(def acc1 (ref 1000 ))
(def acc2 (ref 1000 ))
(defn transfer [from-acct to-acct amt]
  (dosync
    (alter to-acct + amt)
    (alter from-acct - amt)))
```

We use `dosync` to create transactions and `alter` to set the in-transaction value of ref.  

## commute

`commute` is used to update a ref's value within a transaction, just like `alter`. However, 
it behaves different in that it doesn't perform checks and doesn't retry transactions. This 
can boost performance but also introduce bugs. We need to use this only when we are sure that 
a ref can't end up in an invalid state.

Here is an example of when it is better to use `commute`:
```clojure
(def counter (ref 0))

(defn commute-inc! 
  [counter]
  (dosync (Thread/sleep 100) (commute counter inc)))

(defn alter-inc! 
  [counter]
  (dosync (Thread/sleep 100) (alter counter inc)))

(defn bombard-counter!
  [n f counter]
  (apply pcalls (repeat n #(f counter))))

(time (doall (bombard-counter! 20 alter-inc! counter)))
; => Elapsed time: 2007.83 msecs 
; 20 workers * 100 msecs

(dosync (ref-set counter 0))

(time (doall (bombard-counter! 20 commute-inc! counter)))
; => Elapsed time: 301.45 msecs
; we achieved actual concurrency
```

In this example all the `alter` calls step on eachother and cancel transactions, while 
`commute` calls can all be executed without worrying about that. 
