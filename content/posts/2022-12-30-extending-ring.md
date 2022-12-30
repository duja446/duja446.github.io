---
title: "Extending Ring"
date: 2022-12-30T15:32:01+01:00
tags: ["clojure"]
keywords: ["ring", "metosin", "ring-http-response", "muuntaja"]
---

# Extending Ring

Ring stack consists of a chain of middleware function. We can write our own middleware functions or 
we can use libraries that include these middleware functions. There are three popular libraries that 
are used for this purpose:
* `ring-defaults` [https://github.com/ring-clojure/ring-defaults](https://github.com/ring-clojure/ring-defaults)
* `ring-http-response` [https://github.com/metosin/ring-http-response](https://github.com/metosin/ring-http-response)
* `muuntaja` [https://github.com/metosin/muuntaja](https://github.com/metosin/muuntaja)

## ring-defaults

This library provides sensible and secure default configurations of Ring middleware for both websites 
and APIs. It provides: 
* `api-defaults` - support for urlencoded parameters 
* `site-defaults` - support for parameters, cookies, sessions, static resources, file uploads and 
browser security headers
* `secure-api-defaults` and `secure-site-defaults` - same support + forced SSL  

## ring-http-response

This library provides an easier way to write code that returns meaningful responses that map to 
specific HTTP status codes. For example `ok` maps to 200, `found` - 302, and so on.

```clojure
(require [ring.util.http-response :refer :all])

(ok "<p>Hello</p>")
; => {:status 200, :headers {}, :body "<p>Hello</p>"}

(internal-server-error "<p>failed</p>")
; => {:status 500, :headers {}, :body "<p>failed</p>"}

```

## muuntaja

Muuntaja is used for automatic serializing and deserializing of different data formats used in Ring 
applications. These formats include JSON and YAML as weel as Clojure-specific formats such as EDN and 
Transit. To work with this library we just need to include `muuntaja.middleware/wrap-formats` for 
automatic decoding based on request headers.

