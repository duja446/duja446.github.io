---
title: "Clojure - Watches and Validators"
date: 2022-12-26T19:23:58+01:00
tags: ["clojure"]
keywords: ["watches", "validators"]
---

# Watches and Validators

Watches allow us to monitor every atom change and validators allow us to validate 
that change.

## Watches

Watches are just functions that take four arguments: a key, the referene being 
watched, its previous state, and its new state. Here is an example of a watch 
that prints whenever an atom changes its value.
```clojure
(def count (atom 0))

(add-watch count :counter (fn [key atom old-val new-val]
                              (println "Value of " key " has changed from " old-val " to " new-val)))

(swap! count inc)
; => Value of :counter has changed from 0 to 1
; => 1
```

## Validators

Validators specify what states are allowable for a reference. For example here is a validator 
that allows only odd values:
```clojure
(def count (atom 0))

(set-validator! count odd?)

(swap! count inc)
; => 1

(swap! count inc)
; => Invalid reference state
```

