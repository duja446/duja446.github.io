---
title: "Making a Bittorrent Client in Clojure: Introduction"
date: 2023-03-08T21:18:21+01:00
tags: ["clojure", "bittorrent client"]
keywors: ["bittorrent", "clojure", "peer-to-peer", "distributed computing", "file sharing", "open source", "functional programming", "networking", "Java Virtual Machine (JVM)", "protocol", "data streaming", "Java interoperability", "hashing"]
---

# Introduction

Being so fascinated with torrents and peer-to-peer technology I wanted to challenge 
myself and try to fully understand the bittorrent protocol and implement it in Clojure. 
Choosing Clojure for this task presented unique challenges which made me scratch my head 
a couple of times. I can attribute these problems to my lack of Clojure knowledge and the 
lack of helpful material online. To help the Clojure community I wanted to share my 
programming process from start to finish so I can help people who come across similar problems 
solve them without having to bang their head as much as me.

## Skills and Technologies

Here is a list of skill and technologies we will need / be using:
* reading protocol specifications
* using wireshark
* understanding bencode
* hashing
* understanding encoding 
* making tcp clients
* understanding bittorrent messaging
* parsing bitfields

We will be using Clojure (of course) and a little Java interop. For some tasks we will be using 
established libraries and for other we will make our own solutions.

## High Level Overview

Here I want to provide a high level overview of the whole project and list some of the tasks.

1. Analyzing and parsing a `.torrent` file
2. Connecting to 'trackers' and getting the information about the torrent and peers
3. Parsing the tracker response and getting the port and ip of the peers
4. Connecting to peers via TCP 
5. Handshaking with the peers which involves sending a handshake message and getting one back to establish a connection
6. Parsing all the messages transferred from the peer 
7. Writing to a file
 
Some of the tasks listed here are more involved then others but I will try to explain each on to the best of my ability.

## Useful Resources 

Here are some useful resources which will aid us in the creating of this project:
* [BitTorrent specification](https://wiki.theory.org/BitTorrentSpecification#peer_id)
* [Allen Kim's blog post](https://allenkim67.github.io/programming/2016/05/04/how-to-make-your-own-bittorrent-client.html)
* [Clojure documentation](https://clojuredocs.org/)
