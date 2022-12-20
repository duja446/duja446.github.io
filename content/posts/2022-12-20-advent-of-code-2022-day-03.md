---
title: "Advent of Code 2022 - Day 03"
date: 2022-12-20T15:36:35+01:00
tags: ["advent of code 2022"]
---

# Day 03

The algorithm for this challenge is pretty straightforward:
1. Parse the input into list of string
2. Split each string in the middle
3. Find the letter that occurs in both string
4. Convert the letter to a number using the provided method
5. Sum the numbers

This algorithm works for the first part, and needs a little change to solve the second part.
```clojure
(defn to-lines
  "Splits the input into lines"
  [s]
  (string/split s #"\n *"))

(defn split-middle
  "Split a string in the middle"
  [s]
  (let [length (count s)
        midpoint (quot length 2)
        front (subs s 0 midpoint)
        back (subs s midpoint length)]
    [front back]))

(defn appears-in-all
  "Find a letter common to all strings"
  [strings]
  (first (apply set/intersection (map set strings))))

(def priority (zipmap 
                "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
                (range 1 53)))
```

To find a common letter I converted strings into sets that contain only one occurance of each letter. Next I used 
`clojure.set/intersetion` to find all the letters that are in both sets. Then with `first` I get the first one.  
For converting a letter into a number I simply created a map that maps a letter into its value.

# Part 1

For solving the first part we just need to combine all the functions we created and sum the result
```clojure
(defn part1
  [s]
  (->> (to-lines s)
       (map split-middle)
       (map appears-in-all) 
       (map priority)
       (apply +)))
```

# Part 2

For the second part instead of splitting in the middle we need to take triplets of lines and do the same procedure on them
```clojure
(defn part2
  [s]
  (->> (to-lines s)
       (partition 3)
       (map appears-in-all)
       (map priority)
       (apply +)))
``` 
