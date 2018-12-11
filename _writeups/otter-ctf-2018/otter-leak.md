---
title: Otter Leak
contest: OtterCTF 2018
authors: Lucian Nitescu
layout: writeup
---

### Description:

We found out that one of the Otters been leaking information from our network! Find the leaked data.

Format: CTF{flag all uppercase}

[Download](https://nitesculucian.github.io/uploads/otter2/OtterLeak.pcap)

### Solution Author:

Lucian Nitescu, as part of [jmp 0xc0ffee](https://www.google.com/url?q=https://club.securityespresso.org/&sa=D&ust=1544485416463000) team.

### Stats:

200 points / 45 solvers

### Solution:

On this challenge, I was provided with a ```.pcap``` file which contained packets form an internal network. My first step in every pcap file is to lunch ```Network Miner``` tool and take a look at what it retrieves. Here I discovered the following files:

![](https://nitesculucian.github.io/uploads/otter2/image3.png)

Because ```Network Miner``` is a good tool, but not perfect, I had to launch ```Wireshark``` and extract all the files sent by ```10.0.0.6``` host over SMB protocol.

![](https://nitesculucian.github.io/uploads/otter2/image8.png)

Output of the Wireshark file retrieval:

![](https://nitesculucian.github.io/uploads/otter2/image6.png)

At this point, I discovered that all files are of ```.jpg``` extension but contain only one character. Let's read them all at once! 

As you can see in the following image, I got on first row a string which looks like a base64 encoding:

![](https://nitesculucian.github.io/uploads/otter2/image7.png)

I decided to decode my string, and I obtained the following Morse code:

![](https://nitesculucian.github.io/uploads/otter2/image2.png)

I decoded my Morse code and obtained the flag:

![](https://nitesculucian.github.io/uploads/otter2/image4.png)

Added the missing parts from the flag in order to respect the format.

![](https://nitesculucian.github.io/uploads/otter2/image1.png)

![](https://nitesculucian.github.io/uploads/otter2/image5.png)