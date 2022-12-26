---
title: "Clojure - Refs"
date: 2022-12-26T19:38:19+01:00
draft: true
tags: ["clojure"]
keywords: ["refs"]
---

# Refs

Refs are use to manage mutable, shared state like atoms, but they allow us to express that 
an event sould update the state of more than one identity simultaneously. They allow us to 
update the state of multiple identities using transaction semantics. Transaction have three 
features: 
* They are **atomic**, meaning that all refs are updated or none of them are
* They are **consistent**, meaning that refs always have a valid state
* They are **isolated**, meaning that transaction behave as if they are executed serially
