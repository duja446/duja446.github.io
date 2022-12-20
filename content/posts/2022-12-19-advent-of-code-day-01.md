---
title: "Advent of Code 2022 - Day 01"
date: 2022-12-19T15:07:29+01:00
tags: ["advent of code 2022"]
---

# Day 01

This problem boils down to being able to parse the input and being able to sum the total calories for each elf.
For the parsing I used regex to first get all the calories of one elf with regex `\n *\n *` which maches 
anything between two newline characters. Then to convert paragraphs to lines I matched a newline character
with any number of whitespaces.
```clojure
(defn paragraph-lines
  "Splits s intro paragraphs and paragraphs to lines"
  [s]
  (map (#(string/split % #"\n *")) (string/split s #"\n *\n *")))
```

Next I needed to sum calories of one elf which is just parsing and summing all the values from one row. And to
get all the rows I need to just map this function over the whole data structure.
```clojure
(defn sum-row
  "Parses and sums a row"
  [row]
  (->> (map parse-long row)
       (apply +)))

(defn total-elf
  "Gets the total for each elf"
  [s]
  (map sum-row (paragraph-lines s)))
```

## Part 1

For the first part I needed to sort by descending and take the first result.
```clojure
(defn part1
  "Solution for the first part"
  [s]
  (->> (total-elf s)
       (sort-by -)
       (first)))
```

## Part 2

For the second just take the top three results and sum them up
```clojure
(defn part2
  "Solution for the second part"
  [s]
  (->> (total-elf s)
       (sort-by -)
       (take 3)
       (apply +)))
```


