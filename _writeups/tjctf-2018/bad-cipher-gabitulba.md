---
title: Bad Cipher
contest: TJCTF 2018
authors: GabiTulba
layout: writeup
---

## Problem statement:
>My friend insisted on using his own cipher program to encrypt this flag, but I don't think it's very secure. Unfortunately, he is quite good at Code Golf, and it seems like he tried to make the program as short (and confusing!) as possible before he sent it. <br>
> I don't know the key length, but I do know that the only thing in the plaintext is a flag. Can you break his cipher for me? <br>
> [Encryption Program](https://github.com/GabiTulba/TJCTF2018-Write-ups/blob/master/Bad%20Cipher/bad_cipher.py) <br>
> [Encrypted Flag](https://github.com/GabiTulba/TJCTF2018-Write-ups/blob/master/Bad%20Cipher/flag.enc) 
<br><br>

## My opinion:
Again... why the `Reverse Engineering` tag, this time we were given an obfuscated python code that encrypts an input string using a key. **THIS IS CRYPTO!** <br>
Anyway, the vulnerability was that we knew the first bytes of the flag and that gave us a part of the key <br><br>

## Understanding the encryption:
We were supplied with an `encrypted flag=473c23192d4737025b3b2d34175f66421631250711461a7905342a3e365d08190215152f1f1e3d5c550c12521f55217e500a3714787b6554`and some code that encrypted the flag:
```python
message = "[REDACTED]"
key = ""

r,o,u,x,h=range,ord,chr,"".join,hex
def e(m,k):
 l=len(k);s=[m[i::l]for i in r(l)]
 for i in r(l):
  a,e=0,""
  for c in s[i]:
   a=o(c)^o(k[i])^(a>>2)
   e+=u(a)
  s[i]=e
 return x(h((1<<8)+o(f))[3:]for f in x(x(y)for y in zip(*s)))

print(e(message,key))
```
<br>

Firstly, I couldn't bare reading that so I took the time to deobfuscate the code: <br>
```python
message = ""
key = ""


def encrypt(message,key):
	L=len(key)
	s=[message[i::L] for i in range(L)]
	for i in range(L):
		act=0
		enc=""
		for c in s[i]:
			act=ord(c)^ord(key[i])^(act>>2)
			enc+=chr(act)
		s[i]=enc
	return ''.join( hex(ord(y))[2:] for y in ''.join(''.join(x) for x in zip(*s)))

print encrypt(message,key)
```
<br>
Ok, now let's understand the encryption: <br>

**NOTE** : The encryption works properly only if the message's length is a multiple of the key's length (this is due to `zip(*s)`), I'll explain it a bit later.<br>

**NOTE** : `h((1<<8)+o(f))[3:]` and `hex(ord(y))[2:]` give the same output since `(1<<8)` is 256 and that's `0x100` in hex, so `h((1<<8)+o(f))` will always be `0x1XX` <br>

The encryption function takes a key and a message. <br>
It then creates the matrix `s` of size `len(key) x (len(message)/len(key))` which pretty much is the message written like a normal text that moves to a new row when there is no space left on the current row, except the rows and columns are reversed (see the following example). <br>
>`>>> import sys` <br>
>`>>> message='thisismymessage!'` <br>
>`>>> key='Akey'` <br>
>`>>> L=len(key)` <br>
>`>>> s=[message[i::L] for i in range(L)]` <br>
>`>>> for i in range(len(s)):` <br>
>`...     for j in range(len(s[i])):` <br>
>`...         sys.stdout.write(s[i][j])` <br>
>`...     sys.stdout.write('\n')` <br>
>`...` <br>
>`tima` <br>
>`hseg` <br>
>`imse` <br>
>`sys!` <br>

<br>

Now, for every line of the matrix `s` the following process is applyied: <br>

```python
for i in range(L):
	act=0
	enc=""
	for c in s[i]:
		act=ord(c)^ord(key[i])^(act>>2)
		enc+=chr(act)
	s[i]=enc
```

<br>

1. The i-th byte of the key is used in a xor encryption.
2. Each byte of the ciphertext is dependent of the previous byte.
<br><br>

After that, the last line of the `encrypt` function returns the hex of the string `''.join(''.join(x) for x in zip(*s))` which means it reverses the process that generated `s` in the first place, so the message's bytes are in the same order both in ciphertext and plaintext! <br>

**NOTE** : Let's talk about `zip(*s)`, the zip() function stops when the shortest array reaches it's end. So if `len(message)` is not a multiple of `len(key)` the last bytes of the message will be ignored (see the python [documentation](https://docs.python.org/2/library/functions.html#zip)). This was a huge help in solving the problem since now we know now that the key's lenght divides 56.
<br>

## Finding the key length:

Now we know that the key's lenght might be: `1, 2, 4, 7, 8, 14, 28 or 56`, most probably it will be either `4, 7, 8 or 14` so let's try them one by one: <br>

Sice we know the first 6 bytes of the flag: `tjctf{`, we can check 4 by hand and see if it's wrong: <br>

  1. The first byte of the key is `hex(ord('t')^int('47',16))=='0x33' or '3'`
  2. If the key length was 4 then at the 5-th byte `hex(int('2d',16)^ord('3')^(int('47',16)>>2)==0xf` should be `'f'` or `0x66`, so the key is longer than 4.
  3. Now we can find the first 6 bytes of the key: <br>
  
>`>>> x=['47','3c','23','19','2d','47']` <br>
>`>>> x=[int(y,16) for y in x]` <br>
>`>>> flag='tjctf{'` <br>
>`>>> key=''.join(chr(ord(flag[i])^x[i]) for i in range(len(x)))` <br>
> `>>> key` <br>
> `'3V@mK<'` <br>

<br> 

For the other lenghts I wrote a script that will give us a partially decrypted flag for each length: <br>

```python
message = open('flag.enc').read().strip('\n')

def transform(msg):
	out=''
	for i in range(0,len(msg),2):
		out+=chr(int(msg[i:i+2],16))
	return out

def decrypt(msg,l):
	flag='tjctf{'+(l-6)*'\xff'
	key=[]
	for i in range(len(flag)):
		key.append(chr(ord(msg[i])^ord(flag[i])))
	keylen=len(key)
	dec=''
	act=[0 for i in range(keylen)]
	for i in range(len(msg)):
		if(i>=keylen):
			act[i%keylen]=ord(msg[i])^ord(key[i%keylen])^(ord(msg[i-keylen])>>2)
		else:
			act[i]=ord(msg[i])^ord(key[i])
		dec+=chr(act[i%keylen])
	print list(dec)
	

print 'Msg Len:',len(transform(message))

decrypt(transform(message),7)
decrypt(transform(message),8)
decrypt(transform(message),14)
```

<br>

And the output is:<br>

> `Msg Len: 56` <br>
> `['t', 'j', 'c', 't', 'f', '{', '\xff', ' ', '\x02', 's', 'F', 't', ':', '\x9a', 'U', '\x02', 'X', 'W', 'c', '>', '\xce', 'l', '\\', '<', 'd', 'v', '\x17', '\xf2', '\x14', '\r', 'V', 'u', 'D', '#', '\xd2', '\x11', '^', '\\', 'V', '\x17', 'l', '\xc1', '*', '\x03', 'X', '7', '}', 'W', '\x9b', '=', 'u', 'S', '\x00', '8', 'F', '\x88']` <br>
> `['t', 'j', 'c', 't', 'f', '{', '\xff', '\xff', 'y', 'b', 'e', '_', 'W', 'r', '\xa3', '\xbf', '3', 'i', 'n', 'g', '_', 'm', '\xcb', '\x94', '3', 'n', 'c', 'R', 'y', 'p', '\xc6', '\xfa', '0', 'N', '_', 'M', 'Y', '5', '\xf7', '\xa7', 'f', '_', 'W', '4', 'S', 'n', '\xe6', '\x94', 'v', '_', 's', 'm', '4', 'R', '\xa5', '\xb6']` <br>
> `['t', 'j', 'c', 't', 'f', '{', '\xff', '\xff', '\xff', '\xff', '\xff', '\xff', '\xff', '\xff', 'D', '\x1b', '^', 'Z', 'e', '*', '\xd4', '\xbb', '\xa8', '\xb3', '\xdc', '\xf2', '\xc7', '\x89', '\x1c', '\x1b', 'M', 'x', '@', '(', '\xd9', '\xc3', '\xbd', '\xc4', '\xee', '\x9a', '\xb7', '\xa3', ',', '\x13', ']', '>', 'j', 'G', '\x9d', '\xfc', '\x94', '\xd7', '\xa5', '\xa7', '\x98', '\xf7']` <br>

<br>

Ok so it's clear that the key's length is 8.

## Finding the flag:

Now we have a lot of info about the flag: `tjctf{��ybe_Wr��3ing_m˔3ncRyp��0N_MY5��f_W4Sn��v_sm4R��` and we know the key's length, the flag pretty much says `maybe writing my encryption myself wasn't very smart`, so I tried to write it in leet. Eventually I tried `tjctf{m4` as the first 8 bytes and ran [decrypt.py](https://github.com/GabiTulba/TJCTF2018-Write-ups/blob/master/Bad%20Cipher/decrypt.py) and done! <br>

Flag: **tjctf{m4ybe_Wr1t3ing_mY_3ncRypT10N_MY5elf_W4Snt_v_sm4R7}**
