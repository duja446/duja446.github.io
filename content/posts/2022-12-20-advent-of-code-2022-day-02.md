---
title: "Advent of Code 2022 - Day 02"
date: 2022-12-20T15:36:15+01:00
tags: ["advent of code 2022"]
---

# Day 02

To solve this problem I had to first find a score for the input (game of rock, paper, scissors), and then sum them up.
I decided to encode each outcome in a map where the key is the game and the output is the number of points which that 
game awards.
```clojure
(def ROCK       1)
(def PAPER      2)
(def SCISSORS   3)

(def LOSE       0)
(def DRAW       3)
(def WIN        6)

(def scores1 {
              "A X" (+ ROCK DRAW)
              "A Y" (+ PAPER WIN)
              "A Z" (+ SCISSORS LOSE)

              "B X" (+ ROCK LOSE)
              "B Y" (+ PAPER DRAW)
              "B Z" (+ SCISSORS WIN)

              "C X" (+ ROCK WIN)
              "C Y" (+ PAPER LOSE)
              "C Z" (+ SCISSORS DRAW)
              })

```

# Part 1

Then the actual solving is pretty easy just split the input into lines and then map over it with `scores1`. In Clojure we can use
a map as a function that accepts a key as and argument and returns the associated value. If you think about it maps and functions 
are the same, they both just map an input to an output.
```clojure
(defn part1
  "Solves the first part"
  [s]
  (->> (map scores1 (to-lines s))
       (apply +)))
```

# Part 2

For the second part we just change the results map to reflect the changes in the task and do the same.
```clojure
(def scores2 {
              "A X" (+ SCISSORS LOSE)
              "A Y" (+ ROCK DRAW)
              "A Z" (+ PAPER WIN)

              "B X" (+ ROCK LOSE)
              "B Y" (+ PAPER DRAW)
              "B Z" (+ SCISSORS WIN)

              "C X" (+ PAPER LOSE)
              "C Y" (+ SCISSORS DRAW)
              "C Z" (+ ROCK WIN)
              })

(defn part2
  [s]
  (->> (map scores2 (to-lines s))
       (apply +)))

```
