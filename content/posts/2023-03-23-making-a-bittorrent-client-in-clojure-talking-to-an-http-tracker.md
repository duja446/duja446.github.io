---
title: "Making a Bittorrent Client in Clojure: Talking to an HTTP Tracker"
date: 2023-03-23T11:14:31+01:00
draft: true
tags: ["clojure", "bittorrent client"]
keywors: ["bittorrent", "clojure", "peer-to-peer", "distributed computing", "file sharing", "open source", "functional programming", "networking", "Java Virtual Machine (JVM)", "protocol", "data streaming", "Java interoperability", "hashing"]
---

# Torrent Trackers

A torrent tracker is a service which keeps information and statistics and can be queried. A tracker can either be an HTTP tracker 
or and UDP tracker. They differ in that they use different communication protocols but the info they store and relay is still the same. 
For this project we will implement a mechanism which requests info from an HTTP tracker. It is also worth noting that modern torrent clients 
use the *scrape* convention which makes the process of getting information easier, however I will not be implementing that functionality 
for the time being.

## Tracker Request

Lets look at the official specification and see what we need to send to the tracker. There are 6 mandatory fields, namely:
* info_hash
* peer_id
* port
* uploaded
* downloaded
* left

Port, uploaded, downloaded and left don't require little explaining since they can be calculated easily. 
The other two field info_hash and peer_id require our attention as they have some rules on how we calculate them.

### Info Hash

Info hash is a *urlencoded* 20-byte *SHA1* hash of the value of the info key from the `.torrent` file. Let's ignore the *urlencoded* part and 
focus on the *SHA1* hash part. Hash is a function used in cryptography to transform a data of arbitrary size to a fixed-size value. You can look 
at it as a function which takes some data and calculates its signature. This means that same data will produce the same signature and each change in 
data no matter how small causes the hash function to produce a different signature. A *SHA1* hash refers to the underlying algorithm of the hash function, 
in this case the [SHA1](https://en.wikipedia.org/wiki/SHA-1) algorithm. For our purpose we don't need to know how it computes the hash.

Now that we know what a hash is we need to figure out what exactly we need to hash. According to the specification we need to hash the value of the info key. 
In practice this means we don't hash the decoded value of the info field, rather we hash it without doing the encoding. Let's see how we can do this in Clojure: 
For this project I will be using the [Buddy](https://github.com/funcool/buddy) library to do the hashing:

```clojure
(require '[clojure.java.io] '[bencode.core :refer [write-bencode]] '[buddy.core.hash :refer [sha1] '[buddy.core.codecs :refer [bytes->hex]]] )

(defn info-hash
  [metadata]
  (-> (doto (java.io.ByteArrayOutputStream.)
        (write-bencode (get metadata "info")))
      .toByteArray
      sha1))

(info-hash (torrent->data "ubuntu.torrent"))
; [-103, -56, 43, -73, 53, 5, -93, -64, -76, 83, -7, -6, 14, -120, 29,
;  110, 90, 50, -96, -63]

(bytes->hex *1)
; "99c82bb73505a3c0b453f9fa0e881d6e5a32a0c1"
```

We defined a function which takes a clojure map which represents the decoded `.torrent` file. We then get the `info` field from it and encode in back to 
bencode. We now transform it into a byte array and hash it using the `sha1` function. This outputs a byte array which we can use for our purpose. We can also 
transform this into a string to see if we did everything right using the `bytes->hex` function. By going to the [ubuntu download webiste](https://torrent.ubuntu.com/tracker_index) 
we can see that we got the same info hash that is listed with our file.

### Peer Id

Peer id is  a *urlencoded* 20-byte string used as a unique ID for the client, generated by the client at startup. There are two conventions when it comes to 
generating this value, namely Azureus-style and Shadow's-style. We will use the Azureus-style which uses the following encoding: '-', two characters for client id 
four ascii digits for version number, '-', followed by random numbers. I will use `BU` for client id and `0001` for the version number. Let's write a function which 
generates our peer id: 

```clojure
(defn peer-id
  []
  (str "-BU0001-" (apply str (take 12 (repeatedly #(rand-int 9))))))
(peer-id)
; "-BU0001-548563511188"
```

### Other fields

For the `port` we will use `6881`, `uploaded` is going to be `0` since we haven't started downloading, `downloaded` is going the equal the length of the file, 
and `left` is going to be the same as `downloaded`.

### URL encoding

URL encoding is used to convert characters into a representation which can be transmitted over the Internet. For example to transmit the space character we can't 
just input it since that breaks the URL, we need to encode it to `%20`. Since info_hash and peer_id contain characters which may break the URL we need to URL encode them. 
For this purpose we can use the `java.net.URLEncoder` class from Java. It is here that we run into a problem, if we just try to use the `encode` method of the encoder 
as is our data is going to be encoded as `UTF-8` which is a format that doesn't represent bytes literally. This means our encoding will not represent our bytes. We need 
and encoding that represents the bytes literally, luckily we have one and it is `ISO8859_1`. Let's see the difference between them visually:

```clojure
(URLEncoder/encode (String. (info-hash (torrent->data "ubuntu.torrent")))) 
; "%EF%BF%BD%EF%BF%BD%2B%EF%BF%BD5%05%EF%BF%BD%EF%BF%BD%EF%BF%BDS%EF%BF%BD%EF%BF%BD%0E%EF%BF%BD%1DnZ2%EF%BF%BD%EF%BF%BD"
(URLEncoder/encode (String. (info-hash (torrent->data "ubuntu.torrent")) "ISO_8859_1") "ISO_8859_1") 
; "%99%C8%2B%B75%05%A3%C0%B4S%F9%FA%0E%88%1DnZ2%A0%C1"
```

Now that we know how to calculate all the values and encode them let's move on to actually sending the request.

## Sending the Request

To send the request we will use the [`clj-http` library](https://github.com/dakrone/clj-http). Let's see how we can use it to send an HTTP GET request:
```clojure
(require '[clj-http.client :as client])

(client/get "https://google.com")
```

Now we know how to send a GET request but we still need to send our data with it. We do that by sending it as query parameters. As the name suggests they 
are parameters stored in the url. An example would be `http://example.com/articles?sort=ASC&page=2`, here `sort` and `page` would be the keys with values `ASC` and `2`.
The library provides a way to send these by specifying them in the optional parameters of the `client/get` function. But there is a problem with this approach, and it's the 
fact that the library automatically encodes our data which is something we don't want because our `info_hash` is already encoded and encoding it again would mess it up.
So instead of using that way we will code up our own simple function to do this for us.

```clojure
(defn- add-query-params
  [url query-params]
  (reduce 
    (fn [acc [k v]] 
      (let [connector (if (= (last acc) \?) "" "&")]
       (str acc connector (name k) "=" v)))
    (str url "?")
    query-params))
```

Now we can write a function that will take our data and encode it and send the request.

```clojure
(defn query-tracker
  [url info-hash peer-id left & {:keys [port downloaded uploaded] :or {port 6881 downloaded 0 uploaded 0}}]
  (let [encoded-info-hash (urlencode-bytes info-hash)
        url-with-params (add-query-params url 
                                          {:info_hash encoded-info-hash
                                           :peer_id peer-id
                                           :left left
                                           :port port
                                           :downloaded downloaded
                                           :uploaded uploaded})]
    (-> url-with-params
        (client/get {:as :byte-array})
        :body
        read-bencode-string)))
```

Here we just encode our `info_hash` and make our URL by adding our parameters to it. We then send the request and get the response as a `byte-array` so we can process it. 
For a response we get a big map, but we only need the `:body` field so we extract it. Now because the tracker response is a Bencoded dictionary we decode it. 



