---
title: "2022 12 26 Clojure Vars"
date: 2022-12-26T21:09:03+01:00
draft: true
tags: ["clojure"]
keywords: []
---

# Vars

Vars are associations between symbols and objects. They can also be used in tackling concurrency 
tasks. We can dynamically bind them and alter their roots.

## Dynamic Binding

Dynamic vars van be useful for creating a global name that should refer to different values 
in different contexts.

### Creating and Binding Dynamic Vars

```clojure
(def ^:dynamic *email-address* "test@mail.org")

(println *email-address*)
; => test@mail.org

(binding [*email-address* "another@mail.org"]
  *email-address*)
; => another@mail.org
```

### Dynamic Var Uses

Dynamic vars are most often used to name a resource that one or more functions target. For example 
Clojure comes with built-in dynamic vars like `*out*` which represent the standard output for 
print operations.
```clojure
(binding [*out* (clojure.java.io/writer "print-output")]
  (println "Hello, World!"))

(slurp "print-output")
; => Hello, World!
```

