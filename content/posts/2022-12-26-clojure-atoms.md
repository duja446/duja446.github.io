---
title: "Clojure - Atoms"
date: 2022-12-26T19:07:25+01:00
tags: ["clojure"]
keywords: ["atoms", "concurrent", "parallel"]
---

# Atoms

Atoms are a type a reference that can hold a single value. They represent a 
single, indivisible unit of state that can be changed. Atoms are used to manage 
mutable, shared state in concurrent environment.
```clojure
(def count (atom 0))

(@count)
; => 0
```

To update the atom we can use `swap!`. We never update the value inside the atom 
we just make it reference a new value. `swap!` receives an atom and a function 
as arguments and applies that function to the atom's current state to produce 
a new value.
```clojure
(swap! count inc)

(@count)
; => 0
```

Two separate threads can't change the value of an atom at the same time because
`swap!` implements *compare-and-set* semantics meaning it does the following:
1. Reads the current state of the atom
2. Applies the update function to that state
3. Chech whether the value it read in step 1 is identical to the atom's current value
4. If if it, then `swap!` update the atom to refer to the result of step 2 
5. If it isn't, then `swap!` retries, goinf through the whole process again 

`swap!` updates an atom synchronously meaning it blocks the current thread.

When we want to skip the value checking and just force the value to change, we can 
use `reset!`.
```clojure
(reset! count 0)
```

