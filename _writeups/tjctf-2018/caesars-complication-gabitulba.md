---
title: Caesar's Complication
contest: TJCTF 2018
authors: GabiTulba
layout: writeup
---

## Problem statement:
> King Julius Caesar was infamous for his [wordsearch](https://github.com/GabiTulba/TJCTF2018-Write-ups/blob/master/Caesar's%20Complication/ciphertext) solving speed.
<br><br>
## My opinion:
Even if the idea of this challenge was pretty straight forward, I still found it very frustrating.<br>
<br>
## Understanding the challenge:
First of all, it was clear from the challenge's name that the encryption was a [Caesar Cipher](https://en.wikipedia.org/wiki/Caesar_cipher) with some arbitrary shift key, and also the statement explicitly said that the ciphertext was a [Word Search](https://en.wikipedia.org/wiki/Word_search) table, so to search for the flag we just needed some code to check for the string that starts with`tjctf{` and ends with `}` in all 8 directions for every possible key.<br>
<br>

## The code:
```python
mat=[list(x) for x in open('ciphertext').read().split('\n')]

dirx=[0,1,1,1,0,-1,-1,-1]
diry=[1,1,0,-1,-1,-1,0,1]

def Check(x,y,d):
	s=''
	while(x<len(mat) and y<len(mat[x]) and x>0 and y>0):
		s+=mat[x][y]
		x+=dirx[d]
		y+=diry[d]
		if(s.startswith('tjctf{') and s.endswith('}')):
			print 'Possible flag found:',s
			return

def Search():
	for x in range(len(mat)):
		for y in range(len(mat[x])):
			for d in range(8):
				Check(x,y,d)
def Shift():
	for i in range(len(mat)):
		for j in range(len(mat[i])):
			if(mat[i][j]=='{' or mat[i][j]=='}'):
				continue
			elif(mat[i][j]=='z'):
				mat[i][j]='a'
			else: mat[i][j]=chr(ord(mat[i][j])+1)

for i in range(26):
	print 'Key:',i
	Search()
	Shift()
```

## Finding the flag:
The python code gave the following output:
>Key: 0 <br>
>Key: 1 <br>
>... <br>
>Key: 8 <br>
>Possible flag found: tjctf{idesofmarch} <br>
>Key: 9 <br>
>... <br>
>Key: 25 <br>

So the flag was very easy to find:
Flag: **tjctf{idesofmarch}**
