---
title: N Number
contest: ECSC Romanian Quals 2018
authors: trupples
layout: writeup
---

## Part 1 - The Number

> Unfortunately for you, the address is encrypted with a secret number that you must find. The secret is the largest integer N with the properties:
>
> All digits of N are distinct.
> The product of all of N’s neighboring digits is also a part of N.
>
> For example, 241836 verifies these two requirements (but it is not the largest one!)
> Indeed: 2x4=8, 4x1=4, 1x8=8, 8x3=24, 3x6=18.

Because its digits have to be distinct this number can't possibly be greater
than 10 digits. As we're interested in the greatest such number the search can
be done in reverse order, starting at 9876543210 and going down until the first
and greatest valid number is found.

The code to find the number is ridiculously bad - a sort of an inlined
backtracking search that doesn't use numbers at all but rather strings:

```py
def find():
	digits = "9876543210"

	for a in digits:
		for b in digits:
			if b == a:
				continue
			for c in digits:
				if c == b or c == a:
					continue
				for d in digits:
					if d == c or d == b or d == a:
						continue
					for e in digits:
						if e == d or e == c or e == b or e == a:
							continue
						for f in digits:
							if f == e or f == d or f == c or f == b or f == a:
								continue
							for g in digits:
								if g == f or g == e or g == d or g == c or g == b or g == a:
									continue
								for h in digits:
									if h == g or h == f or h == e or h == d or h == c or h == b or h == a:
										continue
									for i in digits:
										if i == h or i == g or i == f or i == e or i == d or i == c or i == b or i == a:
											continue
										for j in digits:
											if j == h or j == g or j == f or j == e or j == d or j == c or j == b or j == a:
												continue
											n = a+b+c+d+e+f+g+h+i+j
											ok=1
											for p in range(9):
												if str(int(n[p]) * int(n[p+1])) not in n:
													ok=0
											if ok==1:
												return n
```

The desired number is 9872305614.

## Part 2 - Deciphering the code

> U2FsdGVkX1/9Mn5FmMWRvC7FYywB005oHQNClUQZ7pmUT2W7SqE2I00vhmGK/gZx

The decoded base64 string begins with `Salted__` which means it is in the
[OpenSSL salted format](http://justsolve.archiveteam.org/wiki/OpenSSL_salted_format).
The code itself doesn't contain any information about the cipher so we have to
guess it. Luckily enough the first cipher listed by `openssl enc -ciphers`,
namely `aes-128-cbc` is the correct one. The password was the base10 ascii
representation of the number we found earlier.

```sh
$ echo U2FsdGVkX1/9Mn5FmMWRvC7FYywB005oHQNClUQZ7pmUT2W7SqE2I00vhmGK/gZx >ciphertext.b64 
$ base64 -d ciphertext.b64 > ciphertext 
$ openssl enc -aes-128-cbc -d -k 9872305614 -in ciphertext -out plaintext
$ cat plaintext
44▒25'42.0"N 26▒03'30.4"E
```

I'm not sure about the non-printable character (probably it's the degree sign)
but rewriting the decrypted coordinates to match the specified format
`ECSC{XX XX XX.XN XX XX XX.XE}` results in the correct flag:
`ECSC{44 25 42.0N 26 03 30.4E}`.
