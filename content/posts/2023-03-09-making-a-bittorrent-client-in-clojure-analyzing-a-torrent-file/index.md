---
title: "Making a Bittorrent Client in Clojure: Analyzing a Torrent File"
date: 2023-03-09T10:57:32+01:00
tags: ["clojure", "bittorrent client"]
keywors: ["bittorrent", "clojure", "peer-to-peer", "distributed computing", "file sharing", "open source", "functional programming", "networking", "Java Virtual Machine (JVM)", "protocol", "data streaming", "Java interoperability", "hashing"]
---

# Torrent files

For the purposes of this tutorial I will be using the Ubuntu 22.10 desktop torrent which you can get at the official website.

A `.torrent` file contains the meta information about the torrent meaning it doesn't contain the actual 
file we want to download rather it provides information about it and how we can go about download it. Before we 
delve deeper into the information stored in the file we need to understand what *exactly* is in it. Opening it in 
a text editor and scrolling to the top reveals a weird yet somewhat familiar structure:

```text
d8:announce35:https://torrent.ubuntu.com/announce13:announce-listll35:https://torrent.ubuntu.com/announceel40:https://ipv6.torrent.ubuntu.com/announceee7:comment29:Ubuntu CD releases.ubuntu.com10:created by13:mktorrent 1.113:creation datei1666283028e4:infod6:lengthi4071903232e4:name30:ubuntu-22.10-desktop-amd64.iso12:piece lengthi262144e6:pieces310680:×öÑ^ã7 LÖ2õ¼h~GÈ$aò# ãÎ²¸Ëì,>ìº-Ï­H
```

Ignoring the binary data at the end, we can see some words and what looks like links. This is because the information we want to 
view is encoded. In our case it is encode using the [Bencode](https://en.wikipedia.org/wiki/Bencode) encoding scheme. We need to know 
how to interpret this encoded data. Luckily we don't need to understand the whole bencode scheme, we can rather focus just on the four types 
it supports. If you want to learn more I wrote a blog article going deeper on what bencode is and how we can read it, 
which you can read [here](https://blog.duja446.com/posts/2023-03-06-decoding-bencode-in-clojure/). 

## Bencode

Here are the four types that bencode supports: 
* **Byte String** - `<string length encoded in base ten ASCII>:<string data>` - "4:spam" represents "spam"
* **Integers** - `i<integer encoded in base ten ASCII>e` - "i400e" represents "400"
* **Lists** - `l<bencoded values>e` - "l4:spam4:eggse" represents "['spam', 'eggs']"
* **Dictionaries** - `d<bencoded string><bencoded element>e` - "d4:spaml1:a1:bee" represents "{'spam' => ['a', 'b']}" 

With this knowledge we can decipher that our file contains a dictionary with a field "announce" with a value of some url. It also 
contains other fields but we will look at them more closely later.

## Translating Bencode to Clojure

To translate this bencode structure we found to a Clojure data structure we will use the [nrepl/bencode library](https://github.com/nrepl/bencode). 
This saves a from writing a lot of boring code to deal with bencode. 

After creating our project we include this library in our project by adding:
```text
[nrepl/bencode "1.1.0"]
```
Then we require it:
```clojure
(ns bufytorrent.core 
  (:require 
    [bencode.core :refer [read-bencode write-bencode]]))
```

Now we can load our file and use this library to look at it. To do this we need to transform our file into a `PushbackInputStream` and pass 
it to the `read-bencode` function. We will create a helper function to do this process for us when we need to convert a file.
```clojure
(defn torrent->data
  [file]
  (-> file
      clojure.java.io/input-stream
      java.io.PushbackInputStream.
      bencode/read-bencode))
```
To convert our file to `PushbackInputStream` we first make an `InputStream` using the built-in function and then convert this to our desired 
structure. You can read about why we do this and what these streams mean on my [bencode blog post](https://blog.duja446.com/posts/2023-03-06-decoding-bencode-in-clojure/).

Here is a part of our decoded structure:
```clojure
{"announce"
 [104, 116, 116, 112, 115, 58, 47, 47, 116, 111, 114, 114, 101, 110,
  116, 46, 117, 98, 117, 110, 116, 117, 46, 99, 111, 109, 47, 97, 110,
  110, 111, 117, 110, 99, 101],
 "announce-list"
 [[[104, 116, 116, 112, 115, 58, 47, 47, 116, 111, 114, 114, 101, 110,
    116, 46, 117, 98, 117, 110, 116, 117, 46, 99, 111, 109, 47, 97,
    110, 110, 111, 117, 110, 99, 101]]
  [[104, 116, 116, 112, 115, 58, 47, 47, 105, 112, 118, 54, 46, 116,
    111, 114, 114, 101, 110, 116, 46, 117, 98, 117, 110, 116, 117, 46,
    99, 111, 109, 47, 97, 110, 110, 111, 117, 110, 99, 101]]],
 "comment"
 [85, 98, 117, 110, 116, 117, 32, 67, 68, 32, 114, 101, 108, 101, 97,
  115, 101, 115, 46, 117, 98, 117, 110, 116, 117, 46, 99, 111, 109],
 "created by"
 [109, 107, 116, 111, 114, 114, 101, 110, 116, 32, 49, 46, 49],
 "creation date" 1666283028,
 "info"
 {"length" 4071903232,
  "name"
  [117, 98, 117, 110, 116, 117, 45, 50, 50, 46, 49, 48, 45, 100, 101,
   115, 107, 116, 111, 112, 45, 97, 109, 100, 54, 52, 46, 105, 115,
   111],
  "piece length" 262144,
  "pieces"
  [-41, -10, -47, 127, 94, -29, -128, 11, 55, 0, 76, -42, 50, -11,
   -100, -68, -102, 13, 104, 126, 71, -56, 36, 97, 28, -14, 35, 32,
   -116, -29, -50, -78, -72, -53, -20, 44, 62, -20, -70, 45, -49, -83,
   72, 10, 69, -14, -23, -9, 107, 7, 116, -68, 85, -71, -77, -61, -82,
   -12, -128, 119, 23, 99, 79, -38, -33, -112, 40, -96, -3, 36, -37,
   111, 20, 102, 5, 36, -119, -115, 93, 69, 120, -118, 71, -54, -126...
```

## File Structure

Now looking at the decode structure we can see our `announce` field and a byte string which represents the url we saw in the file earlier. 
We can also see other fields, but none of them interest us except `info` field. Stored here are the information about the actual file we need to 
download such as how big it is and how many pieces it has. It is worth noting that this field differs when we have single file and multiple files. 
When only a single file is present we have its name and length, but when we have multiple we have a list of files with their own name and length. 

Now that we have a general picture of what structure contains lets take a look at [the bittorrent specification](https://wiki.theory.org/BitTorrentSpecification) 
for a more detailed look. Our file contains the following important keys:
* **info**: describes the file(s) of the torrent
* **announce**: announce URL of the tracker

Info dictionary contains:
* **piece length**: number of bytes in each piece
* **pieces**: all 20-byte SHA1 hash values concatenated together
* **name**: name of the file in single file mode or name of the directory in multi file mode
* **length**: length of the file in single file mode
* **files**: list of file dictionaries each containing its own name and length *only in multi file mode*

We can now decode `.torrent` files and look at their structure. In later parts we will use the metadata we have access to 
to talk to the tracker and request information from it.



