---
title: "Advent of Code 2022 - Day 05"
date: 2022-12-21T21:33:59+01:00
tags: ["advent of code 2022"]
---

# Day 05

The main challenge for this day is manipulating the datastructure which consists of `n` stacks. Having `peek`, `pop` and `conj` functions 
makes this job easier. First we represent the structure as vector of lists of characters. I added an empty list in the first place because 
the 'commands' in the challenge start from 1 not from 0.
```clojure
;    [D]    
;[N] [C]    
;[Z] [M] [P]
; 1   2   3 
(def sample-input 
  ['() '(\N \Z) '(\D \C \M) '(\P)])
```

The parsing of 'commands' is similar to what we have been doing in the previous challenges. The main difference is that I discovered `re-seq` 
that matches all regex and returns the matches in a list. With this function finding all the numbers is as simple as giving the function a regex `\d+`.
```clojure
(defn commands-to-list
  [s]
  (map 
    (comp (partial map parse-long) (partial re-seq #"\d+")) 
    (string/split s #"\n *")))
```

## Part 1

To manipulate the datastructure using recursion we define a loop that takes a source stack, target stack and number of letters to move. We than ask 
if the number of letters to move is positive (indicating we have letters to move) and if it is recuring with source without the top element, target 
with the top element of source and a decreased `n`. If there are no more letters to move we just build back the strucure by associating the stack at 
the from index with the new source and the stack at to index with the new target.
```clojure
(defn move1
  [stacks [n from to]]
  (loop [source (stacks from)
         target (stacks to)
         n      n]
    (if (pos? n)
      (recur 
        (pop source) ; source without the top element
        (conj target (peek source)) ; target with top element of source on top
        (dec n))
      (assoc stacks ; put new stacks in position
          from source
          to target))))
```

I didn't want to parse the stack structure because it is small so I can input it by hand without a problem, and also I am kinda lazy to do that.

For solving the challenge we just need to apply all the commands and get the top element from each stack.
```clojure
(defn part1
  [stacks commands-string]
  (->> (reduce move1 stacks (commands-to-list commands-string))
       (map peek)
       (apply str)
       ))
```

## Part 2

Now we don't move elements one by one, instead we first assemble the list of element to move in `temp` and the move them at the end of recursion.
```clojure
(defn move2
  [stacks [n from to]]
  (loop [source (stacks from)
         temp ()
         n      n]
    (if (pos? n)
      (recur 
        (pop source)
        (conj temp (peek source))
        (dec n))
      (assoc stacks
          from source
          to (into (stacks to) temp)))))
```

Code to solve the second part is the same we just use `move2` instead of `move1`.  
```clojure
(defn part2
  [stacks commands-string]
  (->> (reduce move2 stacks (commands-to-list commands-string))
       (map peek)
       (apply str)
       ))
```

