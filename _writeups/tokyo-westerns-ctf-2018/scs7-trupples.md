---
title: scs7 
contest: TokyoWesterns CTF 4th 2018
authors: trupples
layout: writeup
---

## Proof of Flag
```
TWCTF{67ced5346146c105075443add26fd7efd72763dd}
```

## Summary
Base59 with a shuffled alphabet. We can figure out the alphabet by sending it
bytes from 59 to 117, observing the last encrypted character, and then using the
observed alphabet to decrypt the flag.

## Proof of solving
I initially tried sending `TWCTF{` + each character and seeing which one would
match the largest prefix. The problem with that is that as they use base59, the
symbol boundaries don't really line up with encodings with power-of-two symbols
so I got many false positives and false negatives.

Here’s the successful solution:

I tried sending all characters from 0 to 99 and seeing the result:

```py
from pwn import *

def request(plaintext):
    r.recvuntil("message: ")
    r.sendline(plaintext)
    r.recvuntil("ciphertext: ")
    return r.recvline()[:-1]

r = remote("crypto.chal.ctf.westerns.tokyo", 14791)
requestcount = 0
r.recvuntil("encrypted flag: ")
encflag = r.recvline().strip()
print("Target flag: \n" + encflag)

for i in range(0, 256):
    print(i, request(chr(i)))
```


And that made the encryption algorithm obvious. Here's an example output of that
program:

```
Target flag:
0KhpGLHUSc3D7B449JsxM4L7etCnYW9HH5jBpo72kPgrgXeWzbKxbhyrkN29gb64
(0, '')
(1, 'S')
(2, 'm')
(3, 'h')
...
(38, 'x')
(39, 'f')
(40, 'E')
...
(57, 's')
(58, '6')
(59, 't')
(60, 'ST')
(61, 'SS')
(62, 'Sm')
(63, 'Sh')
...
(97, 'Sx')
(98, 'Sf')
(99, 'SE')
```

It is clear that the last letter repeats every 59 values. We could call this a
base 59 encoding, but the symbols are not in alphabetical order so the output
changes every time. It's safe to assume that the alphabet is shuffled for every
connection.

In this case we only need to observe 59 encryptions to reconstruct the alphabet
and decrypt the flag:

```py
from pwn import *

r = remote("crypto.chal.ctf.westerns.tokyo", 14791)
r.recvuntil("encrypted flag: ")
encflag = r.recvline().strip()
print("Target flag: \n" + encflag)

alphabet = ["?"] * 59

for i in range(60, 60+59):
    r.recvuntil("message: ")
    r.sendline(chr(i))
    r.recvuntil("ciphertext: ")
    ct = r.recvline().strip()
    alphabet[i % 59] = ct[-1]
    print(i-60)    # to know the progress

plaintext_num = 0
for c in encflag:
    plaintext_num = plaintext_num * 59 + alphabet.index(c)

print(hex(plaintext_num))

# => 0x54574354467b363763656435333436313436633130353037353434336164643236666437656664373237363364647d
# => TWCTF{67ced5346146c105075443add26fd7efd72763dd}
```
