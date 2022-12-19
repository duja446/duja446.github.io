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

To solve the challenge I needed to sort by descending and for the first part take the first result and for the second
part take top three results and sum them.
```clojure
(defn part1
  "Solution for the first part"
  [s]
  (->> (total-elf s)
       (sort-by -)
       (first)))

(defn part2
  "Solution for the second part"
  [s]
  (->> (total-elf s)
       (sort-by -)
       (take 3)
       (apply +)))
```


