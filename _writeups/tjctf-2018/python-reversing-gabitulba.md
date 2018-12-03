---
title: Python Reversing
contest: TJCTF 2018
authors: GabiTulba
layout: writeup
---

## Problem statement:
> Found this flag checking file and it is quite vulnerable <br>
> [Source](https://github.com/GabiTulba/TJCTF2018-Write-ups/blob/master/Python%20Reversing/source.py) <br>
<br>

## My opinion:

I still don't understand why this problem was tagged as `Reverse Engineering`, it was a simple cryptography problem with a classic attack, in my opinion it was way easier than Caesar's Complication, the number of solves talk for themselves.<br><br>

## Understanding the encryption

We were given the following code:<br>
```python
import numpy as np

flag = 'redacted'

np.random.seed(12345)
arr = np.array([ord(c) for c in flag])
other = np.random.randint(1,5,(len(flag)))
arr = np.multiply(arr,other)

b = [x for x in arr]
lmao = [ord(x) for x in ''.join(['ligma_sugma_sugondese_'*5])]
c = [b[i]^lmao[i] for i,j in enumerate(b)]
print(''.join(bin(x)[2:].zfill(8) for x in c))

# original_output was 1001100001011110110100001100001010000011110101001100100011101111110100011111010101010000000110000011101101110000101111101010111011100101000011011010110010100001100010001010101001100001110110100110011101
```
Ok, let's see what's happening step by step:
  1. The `flag` is transformed into an array called `arr`.
  2. Then another array `other`of the same length as `arr` with random values from 1 to 4 is generated using a constant seed (clearly a bad idea since now the encryption is [deterministic](https://en.wikipedia.org/wiki/Deterministic_encryption)).
  3. The array `arr` is multiplied with the array `other` by `arr[i]*=other[i]`.
  4. Then the array `b=arr` is xored element by element with the array `lmao`.
  5. The encrypted flag is the concatenation of the binary representation of every value in array `c=b`.
<br>

One thing that makes this problem a little harder (the encryption isn't easily reversable) is that when the array `arr` is multiplied with the array `other`, in some cases the result is bigger than 256. I saw that when I saw that the length of the output wasn't divisible by 8:
<br>

> `>>> <br>`
> `x='1001100001011110110100001100001010000011110101001100100011101111110100011111010101010000000110000011101101110000101111101010111011100101000011011010110010100001100010001010101001100001110110100110011101'` <br>
> `>>> len(x)%8` <br>
> `2` <br>
<br>

But still, since we know every variable (including `other`) we can ecnrypt, so we can guess the flag character by character by encrypting every possible byte and checking if it matches the desired output, this is known as a [Chosen-plaintext attack](https://en.wikipedia.org/wiki/Chosen-plaintext_attack).

## Decrypting the flag:
Here's the code: <br>
```python
import numpy as np

enc='1001100001011110110100001100001010000011110101001100100011101111110100011111010101010000000110000011101101110000101111101010111011100101000011011010110010100001100010001010101001100001110110100110011101'
flag= ''

def encrypt(flag):
	np.random.seed(12345)
	arr = np.array([ord(c) for c in flag])
	other = np.random.randint(1,5,len(arr))
	arr = np.multiply(arr,other)
	b = [x for x in arr]
	lmao = [ord(x) for x in ''.join(['ligma_sugma_sugondese_'*5])]
	c = [b[i]^lmao[i] for i,j in enumerate(b)]
	y=(''.join(bin(x)[2:].zfill(8) for x in c))
	p=0
	while(p<len(y) and enc[p]==y[p]):
		p+=1
	return p
L=0
while(L<len(enc)):
	for j in range(256):
		if(L<=encrypt(flag+chr(j))-8): #at least 8 new bits have to mach the output
			flag+=chr(j)
			L=encrypt(flag)
			print L
			break
print flag
```
<br>

**NOTE** : It just happened for that my solution worked, since there's not necesarily a one to one mapping to plaintext-ciphertext.
Flag: **tjctf{pYth0n_1s_tr1v14l}**
