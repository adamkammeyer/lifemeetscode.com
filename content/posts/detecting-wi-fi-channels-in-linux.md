---
title: "Detecting Wi-Fi Channels in Linux"
date: 2017-03-09T08:34:28-06:00
draft: false
type: "post"
tags:
  - bash
  - linux
  - network
aliases:
  - /blog/detecting-wi-fi-channels-in-linux
---

Recently I've been diagnosing some WI-FI connection issues at home and needed a way to find what channels were being used. In the past, I've used an app on an old Android phone I had lying around, but decided I needed a more practical way. So I set off to DuckDuckGo for an answer.

It didn't take long to find a couple of nice command-line solutions.

First, find the name of your wireless adapter using

```
$ ip link
```

or

```
$ ifconfig -s
```

Mine was wlo1. Once you have that, you can pass it to the iwlist command. You need to run iwlist with sudo in order to see all channels. Without it you will only see the channels of your WI-FI network.

```
$ sudo iwlist wlo1 scan | grep \Frequency
```

You'll see output similar to:

```
                    Frequency:2.447 GHz (Channel 8)
                    Frequency:2.447 GHz (Channel 8)
                    Frequency:2.412 GHz (Channel 1)
                    Frequency:2.412 GHz (Channel 1)
                    Frequency:2.412 GHz (Channel 1)
                    Frequency:2.412 GHz (Channel 1)
                    Frequency:2.412 GHz (Channel 1)
                    Frequency:2.412 GHz (Channel 1)
                    Frequency:2.412 GHz (Channel 1)
                    Frequency:2.437 GHz (Channel 6)
                    Frequency:2.437 GHz (Channel 6)
                    Frequency:2.437 GHz (Channel 6)
```

If you'd like to get an easy-to-ready breakdown of the number of connections each used channel has, you can pipe the output of iwlist into some additional Bash commands.

```
$ sudo iwlist wlo1 scan | grep \Frequency | sort | uniq -c | sort -n
```

You'll see output simiar to:

```
      1                     Frequency:2.457 GHz (Channel 10)
      2                     Frequency:2.427 GHz (Channel 4)
      4                     Frequency:2.412 GHz (Channel 1)
      4                     Frequency:2.437 GHz (Channel 6)
```

References

* [How to Find The Best Wi-Fi Channel For Your Router on Any Operating System - How-to Geek](https://www.howtogeek.com/197268/how-to-find-the-best-wi-fi-channel-for-your-router-on-any-operating-system/)
* [How to Find the Best WiFi Channel For Your WiFi Network - Make Tech Easier](https://www.maketecheasier.com/find-best-wifi-channel/)
