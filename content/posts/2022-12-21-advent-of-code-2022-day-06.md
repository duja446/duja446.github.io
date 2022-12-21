---
title: "Advent of Code 2022 - Day 06"
date: 2022-12-21T19:15:54+01:00
tags: ["advent of code 2022"]
---

# Day 06

This challenge invloved getting every possible sequence of four adjacent characters and seeing if the characters 
are distinct. This invloved creating a recursive function because there was no built in function to do the task.

## Part 1

We introduce a variable `j` which starts at 4 and then take a substring of the main string from `j - 4` to `j` to 
get the four adjacent characters. We then expland this structure with `apply` and check if all the elements are 
distint with `distinct?`. If they are we found the position, and if not we recur with incresed `j`.
```clojure
(defn part1
  [s]
  (loop [j 4]
    (let [packet (subs s (- j 4) j)]
      (if (apply distinct? packet)
        j
        (recur (inc j))))))
```

## Part 2 

For this part we just need to change the number of adjacent character from 4 to 14.
```clojure
(defn part2
  [s]
  (loop [j 14]
    (let [packet (subs s (- j 14) j)]
      (if (apply distinct? packet)
        j
        (recur (inc j))))))
```

