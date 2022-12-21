---
title: "Advent of Code 2022 - Day 04"
date: 2022-12-21T19:14:55+01:00
tags: ["advent of code 2022"]
---

# Day 04

This challenge didn't involve a whole lot of programming just some clever math. To solve it we first parse the input so 
each row is a vector of four numbers. We then filter this structure by the needs of the challenge
```clojure
(defn number-list
  [s]
  (map 
    (comp (partial map parse-long) #(string/split % #"[-,]")) 
    (string/split s #"\n *")))
```

First split the rows and then split numbers which are divided by `-` or `,`. I also included a step of converting the strings 
to numbers for easier use later.

## Part 1

For one range to be contained in another it start must be greater and its end must be less. We need to cover cases when the first is contained 
in second, and when the second is contained in the first
```clojure
; a b c d
; 2 6 3 4
; a(2) <= c(3) <= d(4) <= b(6)
(defn fully-contains?
  [[a b c d]]
  (or 
    (<= a c d b)
    (<= c a b d)))
```

For the solution we filter with this function and count the elements.
```clojure
(defn part1
  [s]
  (->> (number-list s)
       (filter fully-contains?)
       (count)))
```

## Part 2

For the second part we need to find if they overlap. For this I first thought up how to check if they don't overlap and just took the `complement` 
of that function to see if they do overlap.
```clojure
(defn no-overlap?
  [[a b c d]]
  (or 
    (< b c)
    (< d a)))

(defn part2
  [s]
  (->> (number-list s)
       (filter (complement no-overlap?))
       (count)))
```
