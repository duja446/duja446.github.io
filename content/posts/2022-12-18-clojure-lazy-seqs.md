---
title: "Clojure - Lazy Seqs"
date: 2022-12-18T09:11:56+01:00
tags: ["clojure"]
---

# Lazy Seqs

Many functions that work on seqs return lazy seqs. Members of a lazy seq aren't computed until they
are needed. This has a benefit of making a program more efficient and allowing programmers to 
construct infinite sequences.

## Efficiency

```clojure
(def database 
  {0 {:name "Bill" :age 40 :has-job? true}
   1 {:name "Anna" :age 20 :has-job? false}
   2 {:name "Dave" :age 17 :has-job? true}
   3 {:name "Joe"  :age 29 :has-job? false}})

(defn get-details
  [social-security-number]
  (Thread/sleep 1000)
  (get database social-security-number))

(defn can-hire?
  [record]
  (and (not (:has-job? record))
       (> (:age record) 18)
       record))

(defn identify-potential-hire
  [social-security-numbers]
  (filter can-hire? 
          (map get-details social-security-numbers)))

(time (get-details 0))
; => "Elapsed time: 1001.051 msecs"
; => {:name "Anna", :age 20, :has-job? false}
```

Now if `map`  wasn't lazy it would need to process every member of `social-security-numbers` before passing 
it to `filter`. If we had one million entries in our database this would take one milion seconds, or 12 days.  
But luckily `map` is lazy so it doesn't need to do that. 
```clojure
(time (def mapped-details (map get-details (range 0 1000000))))
; => "Elapsed time: 0.037 msecs"
; => #'user/mapped-details'
```
```clojure
(time (first mapped-details))
; => "Elapsed time: 32020.812 msecs"
; => {:name "Anna", :age 20, :has-job? false}
```

You would think that getting one entry from our lazy seq would take 1 second but it actually took 32 seconds.
This is because Clojure **chunks** lazy seqs meaning Clojure will prepare the next 31 elements when getting one.

## Infinite sequences

We can use `repeat` to create a sequence whose every member is an argument you pass.  
```clojure
(take 5 (repeat "x"))
; => ("x" "x" "x" "x" "x")
```
We can use `repeatedly` to create a sequence which will call the provided function to generate 
each element in the sequence.
```clojure
(take 3 (repeatedly (fn [] (rand-int 10))))
; => (7 1 0)
```

