---
title: "Clojure - Data Structures"
date: 2022-12-16T18:03:56+01:00
keywords: ["clojure"]
---

# Data Structures

Clojure data structures are immutable meaning they can't be changed in place
like in many other programming languages.

## Numbers

Clojure has all the basic number representation found in other programming 
languages plus its own *Ratio* type. Ratio represents a division of integers
that can't be reduced to an integer.
```
92
1.8
1/3
```

## Strings

Strings can only be creted by using double quotes. Concatenation is done
using the `str` function
```clojure
(str "Hello," " world!")
```

## Maps

Map is a collection that maps keys to values. Clojure has two different map types
a hashed and a sorted map.
```clojure
{:first-name "Duja"
 :last-name "446"}
```

The function `get` is used to look up values. We can also treat the map as a function
with the key as its argument to look up values
```clojure
(get {:a 0 :b 1} :a)
; => 0

({:a 0 :b 1} :b)
; => 1
```

## Keywords

Keywords are symbolic indentifiers that evalute to themselves. They can also be used
as function that look up corresponding value in a data structure.
```clojure
{:a {:a 1 :b 2 :c 3}}
; => 1
```

## Vectors

Vectors are similar to arrays being indexed by contiguous integers.
```clojure
(get [3 2 1] 0)
; => 3

(get ["duja" 1 {:data "bbcc"}] 2)
; => {:data "bbcc"}
```

The `conj` function is used to add element to the end of the vector.
```clojure
(conj [1 2 3] 4)
; => [1 2 3 4]
```

## Lists

Lists are similar to vectors beaing a linear list of values. However you can't 
retrieve list values with `get` and `conj` puts the element at the front of the list.
```clojure
'(1 2 3 4)
; => (1 2 3 4)

(conj '(1 2 3) 4)
; => (4 1 2 3)
```

## Sets

Sets are like maps, the only difference being that sets hold only unique values.
```clojure
(hash-set 1 1 2 2)
; => #{1 2}

(:a #{:a 1 :b 2})
; => 1

(get #{:a 1 :b 2 :c 3} :c)
; => 3
```
