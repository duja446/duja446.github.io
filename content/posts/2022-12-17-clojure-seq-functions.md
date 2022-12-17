---
title: "Clojure - Seq Functions"
date: 2022-12-17T18:32:42+01:00
tags: ["clojure"]
---

# Seq Functions

Clojure's seq library has functions that work on Clojure's sequence
abstracion.

## map

Besides the main use of `map` it can also be used by taking multiple 
collections as arguments and taking a collection of functions as an argument.
```clojure
(map str ["a" "b" "c"] ["A" "B" "C"])
; => ("aA" "bB" "cC")
```
When multiple collections are passed to `map` the elements of first collections
will be passed as the first argument of the mapping function, second collection
as second argument and so on.
```clojure
(def sum #(reduce + %))
(def avg #(/ (sum %) (count %)))
(defn stats
  [numbers]
  (map #(% numbers) [sum count avg]))

(stats [3 4 10])
; => [17 3 17/3]
```
This example show how map can be used to iterate through list of functions and
apply them.

## reduce

Two not so obious uses for `reduce`.
First is to use it to transform values of a map, producing a new map with same
keys but updated values.
```clojure
(reduce (fn [new-map [key val]]
          (assoc new-map key (inc val)))
        {}
        {:max 30 :min 10})
; => {:max 31 :min 11}
```
Another is to use reduce to filter keys from a map based on their values.
```clojure
(reduce (fn [new-map [key val]]
          (if (> val 4)
            (assoc new-map key val)
            new-map))
        {}
        {:a 4.2 :b 5 :c 3})
; => {:a 4.2 :b 5}
```

## take, drop, take-while, and drop-while

`take` and `drop` both take a number and a sequence, returning first `n` 
elements or a sequences with first `n` elements removed 
```clojure
(take 3 [1 2 3 4 5])
; => (1 2 3)

(drop 3 [1 2 3 4 5])
; => (4 5)
```

`take-while` and `drop-while` take a *predicate functions* - functions that return either true or false,
to determine when it should stop taking or dropping.
```clojure
(def mood-calendar
  [{:month 1 :mood "happy"}
   {:month 2 :mood "sad"}
   {:month 3 :mood "depressed"}
   {:month 4 :mood "very happy"}])

(take-while #(< (:month %) 3) mood-calendar)
; => ({:month 1, :mood "happy"}
;     {:month 2, :mood "sad"})

(drop-while #(< (:month %) 3) mood-calendar)
; => ({:month 3, :mood "depressed"}
;     {:month 4, :mood "very happy"})
```

## filter and some

`filter` returns all elements of a sequence that test true for a predicate function. It has a similar use
case to `take-while` and `drop-while`, but `filter` can end up processing all data which is not always necessary.

`some` function return the first truthy value returned by a predicate function. 

## sort and sort-by

`sort` is used to sort elements in asceding order.

If the sorting is more complicated we can use `sort-by` sort elements using a function. 
```clojure
(sort-by count ["aaa" "bb" "c"])
; => ("c" "bb" "aaa")
```

## concat

`concat` appends the members of one sequence to the end of another. 
```clojure
(concat [1 2] [3 4])
; => (1 2 3 4)
```

