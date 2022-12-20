---
title: "Clojure - Namespaces"
date: 2022-12-19T20:36:19+01:00
tags: ["clojure"]
---

# Namespaces

Namespaces contain maps between between human-friendly *symbols* and references known as *vars*.
Clojure allows for chaning of namespaces and creating new ones.

To create a namespace we can use the function `create-ns`, the function `in-ns` and the macro `ns`.
`create-ns` takes a symbol and creates a namespace, `in-ns` does that and moves into the created  
namespace.

## refer

`refer` allows us to bring stuff from other namespaces into the current one. If called on its 
own it brings everything from the namespace, but it can also be called with the filters `:only`,
`exclude` and `rename`.
We can also define *private* functions which can't be brought to other namespaces using `defn-`
```clojure
(in-ns 'space.private)
(defn- private-function
  [])

(in-ns 'space.other)
(space.private/private-function) 
(refer 'space.private :only ['private-function])
```

The last two lines would throw an error.

## alias

`alias` allows us to shorten a namespace name for using fully qualified symbols. 
```clojure
(clojure.core/alias 'new 'space.new)
(new/some-fn)
```

## ns macro

The `ns` macro is used when working with namespaces in files because it provides many 
functionalities. You can use *references* with `ns` to use different functions of the macro. 
The six possible references are:
- `(:refer-clojure)` 
- `(:require)` 
- `(:use)` 
- `(:import)` 
- `(:load)` 
- `(:gen-class)` 

`(:require)` works like the `require` function just with different syntax. You can require 
any number of files and use `:as` to alias them to different names. `:refer` can be used 
in the same way as the `refer` function.
```clojure
(ns exmaple.core
  (:require [clojure.string :as string]
            [clojure.set :refer :all]))
```

