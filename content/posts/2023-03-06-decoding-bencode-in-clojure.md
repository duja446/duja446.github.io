---
title: "Decoding Bencode in Clojure"
date: 2023-03-06T07:54:35+01:00
tags: ["clojure"]
keywords: ["bencode", "java", "pushback"]
---

# Preface

Majority of this code is taken from [nREPL's repository](https://github.com/nrepl/bencode). This article aims to provide more of 
an explanation behind the code and ditches the netstring part of the repository.

# What is Bencode

Bencode is an encoding scheme used in the BitTorrent protocol when encoding metadata. This bencoded metadata is stored 
in `.torrent` files. 

Bencode supports four types of values:
* byte strings
* integers
* lists 
* dictionaries 

Learn more: 
[Wikipedia](https://en.wikipedia.org/wiki/Bencode)
[BitTorrent Specification](https://wiki.theory.org/BitTorrentSpecification#Bencoding)

## Reading bencode in Clojure

Bencode is tightly packed meaning there are no separators between structures, they are all packed like a single 
byte string. Because of that we need to 'consume' the byte string as we go along it. This means that we need to 
read a certain byte or bytes and then move past them. To accomplish this we will use [Java Input Streams](https://docs.oracle.com/javase/7/docs/api/java/io/InputStream.html) 
mainly [Pushback Input Stream](https://docs.oracle.com/javase/7/docs/api/java/io/PushbackInputStream.html) which 
is a variant of an input stream.

You may come across two types of Java classes that consume files, [readers](https://docs.oracle.com/javase/7/docs/api/java/io/Reader.html) and [input streams](https://docs.oracle.com/javase/7/docs/api/java/io/InputStream.html) 
. The difference between them is that readers are used when we need to deal with Unicode character data, and input streams are used for 
dealing with binary data. Since bencode produces binary data we need to use an input stream rather than a reader.

### Reading bytes

Before we can read bytes we need to figure out how we can read a single byte. To 
do that we can use the [read](https://docs.oracle.com/javase/7/docs/api/java/io/InputStream.html#read()) method of our 
input stream instance. This method reads a byte and returns it or `-1` indicating a fail. 

```clojure
(defn read-byte
  [input]
  (let [c (.read input)]
    (when (neg? c)
      (throw (Exception. "Invalid byte read")))
    c))
```

To read multiple bytes we can once again use the read method, but instead of providing no arguments we provide 
it with with a buffer, offset and a length so we can read `length` number of bytes at an offset and put it 
into our buffer. We first create a byte array by using `(byte-array n)`, then read the bytes and put them in our 
buffer. We also need to handle the case when this fails and throw an exception.

```clojure
(defn read-bytes 
  [input n]
  (let [content (byte-array n)
        result (.read input content 0 n)]
    (when (neg? result)
      (throw (Exception. "Invalid bytes read")))
    content))
```

To know how many bytes we need to read, we read an integer before the colon. For example if we have a bencoded 
byte string `4:duja`, we first need to parse the `4` before the colon to know that we need to read the next 4 bytes 
to get the full byte string.

### Reading size numbers

These numbers differ in the way we read them from the integers which we will see how to read later. Because we 
read a number digit by digit we need to come up with a way to transform the read digits into a number. We do this 
by recuring until we hit a delimiter character (in our case the colon character). We multiply our current number with `10` 
to make space for our new digit, and then add our digit to it. We also need to transform our byte to a digit which we do 
by subtracting `48` from it because numbers start from `48` in the ASCII table.

```clojure
(defn read-long
  [input delim]
  (loop [n (long 0)]
    (let [b (read-byte input)]
      (cond 
        (= b delim) n 
        :else (recur (+ (* n (long 10)) (- (long b) (long 48))))))))
```

### Reading a byte string

We are now ready to read a byte string. We just combine our `read-bytes` function with our `read-long` function.

```clojure
(def colon 58)
(defn read-bytestring 
  [input]
  (read-bytes input (read-long input colon)))
```

### Reading a bencode token

To read a bencode token we need to know what type it is. As mentioned at the beginner we have four options here: a bytestring, 
integer, list and a dictionary. Every type except a bytestring has a starting character and ends with `e` character. Here are examples 
of these types taken from the wiki:
* integer - `i4e`  represents `4` 
* list - `l4:spam4:eggse`  represents `["spam", "eggs"]` 
* dictionary - `d3:cow3:moo4:spam4:eggse`  represents `{ "cow" => "moo", "spam" => "eggs" }` 

Now checking what is a type of a bencode token is pretty simple. We just check what are read byte is and act accordingly. 
We also need to handle a reading of a bytestring which we know doesn't start with any characters. When we don't read any special 
character we know we hit a byte string but our file pointer `consumed` the first byte of that bytestring. To reverse this we use `unread` 
method of our input stream to put our lost byte at the front of the input stream. Then we just continue reading as a bytestring.

```clojure
(def e 101)
(def i 105)
(def l 108)
(def d 100)
(defn read-token 
  [input]
  (let [ch (read-byte input)]
    (cond 
      (= e ch) nil
      (= i ch) :integer
      (= l ch) :list
      (= d ch) :map 
      :else (do 
              (.unread input (int ch))
              (read-bytestring input)))))
```

### Reading bencode

To read bencode we put together our token function with functions that read the bencode types. We now need to make 
those functions.

```clojure
(declare read-integer read-list read-map)

(defn read-bencode
  [input]
  (let [token (read-token input)]
    (case token 
      :integer (read-integer input)
      :list (read-list input)
      :map (read-map input)
      token)))
```

### Reading integers, lists and dictionaries

Because we didn't hardcode the colon as a delimiter we can re purpose our `read-long` to read an integer by 
providing `e` as a delimiter.

```clojure
(defn read-integer
  [input]
  (read-long input e))
```

For reading lists and dictionaries we need to be able to read a sequence of bytestring. This is just reading 
a bytestring repeatedly until we read `e` which returns `nil` in our `read-token` function.    

```clojure
(defn token-seq
  [input]
  (->> #(read-bencode input)
       repeatedly
       (take-while identity)))
```

Reading a list just means reading a sequence of tokens and putting it into a vector.

```clojure
(defn read-list
  [input]
  (vec (token-seq input)))
```

Reading a dictionary is a little more complicated. We also read a sequence and then partition 
all pairs because they represent a key value pair. Then we just transform the bytes of the key to 
a string.

```clojure
(defn read-map
  [input]
  (->> (token-seq input)
       (into {} (comp (partition-all 2)
                      (map (fn [[k v]]
                             [(String. k "UTF-8") v]))))
```






