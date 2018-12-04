---
title: Nothing But Everything
contest: TJCTF 2018
authors: GabiTulba
layout: writeup
---

## Problem statement:
>My computer got infected with ransomware and now none of my [documents](https://github.com/GabiTulba/TJCTF2018-Write-ups/blob/master/Nothing%20But%20Everything/7459b0c272ba30c9fea94391c7d7051d78e1732c871c3a6f27070fcb34f9e734_encrypted.tar.gz) are accessible anymore! If you help me out, I'll reward you a flag!
<br><br>

## My opinion:
This problem was easy yet pretty fun. Of course the 'ransomware' dindn't provide any security at all, but it would still be a fun farce to play on a friend.<br><br>

## Finding out the encryption:
There are a few clues that reveal the encryption mechanism:
  1. Both the names and the contents were encrypted.
  2. There is no Private Key?Public Key pair involved, so the encryption system is simple and most probably [deterministic](https://en.wikipedia.org/wiki/Deterministic_encryption) and easily reversible.
  3. The file names/ directory names and contents of the files are all numbers (very long numbers). 
  4. The file and directory names varied quite a bit in length.
<br>
This lead me to think that the everything was transformed somehow byte by byte.<br>
I then tought that the process was similar to how text messages usually are transformed to integers during RSA encryption so I tried that with the main directory's name.<br>

> `>>> x=1262404985085867488371`<br>
> `>>> x=hex(x)[2:].strip('L')`<br>
> `>>> x=''.join([chr(int(x[i:i+2],16)) for i in range(0,len(x),2)])`<br>
> `>>> print x`<br>
> `Documents`<br>
<br>

**Bingo!** <br>
Now we know the encryption mechanism so we just need to write some clever script that decrypts everything.<br><br>

## Decrypting the files:
I chose python's [os](https://docs.python.org/2/library/os.html) module, and a [DFS](https://en.wikipedia.org/wiki/Depth-first_search) algorithm to decrypt the files, I put the extracted archive in a folder named `Encrypted` which is in the same directory as the script, the output is the folder named `Decrypted`.<br>
Since the code is self explanatory, I won't explain it any further:
```python
import os

def Join(path,directory):
	return path+'/'+directory

#decrypt a string
def dec_str(filename):
	f=hex(int(filename)).strip('L')[2:]
	return ''.join(chr(y) for y in [int(f[i:i+2],16) for i in range(0,len(f),2)])

#decrypt a file's contents and name
def dec_file(filename):
	if(filename=='HAHAHA.txt'):
		return
	decfilename=dec_str(filename)
	os.rename(filename,decfilename)

	f=open(decfilename,'r')
	content=f.read()
	f.close()
	deccontent=dec_str(content)
	open(decfilename,'w').write(deccontent)

#decrypt the name of a file and rename it
def dec_filename(filename):
	decfilename=dec_str(filename)
	os.rename(filename,decfilename)	

#DFS for decrypting everyting in a directory
def DFS(path):
	father=os.getcwd()
	os.chdir(path)

	l=os.listdir(os.getcwd())
	for name in l:
		if os.path.isdir(Join(path,name)):
			dec_filename(name)
			DFS(Join(path,dec_str(name)))
		else:
			dec_file(name)
	
	os.chdir(father)

for y in os.listdir(os.getcwd()):
	if(os.path.isdir(Join(os.getcwd(),y))):
		DFS(Join(os.getcwd(),y))
		os.rename(y,'Decrypted')
```

<br>

## Finding the flag:
This part was very easy, I simply opened every file (mostly out of curiosity) until I found the flag. It was in `here (2).xlsx`, as the name of the sheet:<br>
![Flag](https://github.com/GabiTulba/TJCTF2018-Write-ups/blob/master/Nothing%20But%20Everything/flag.png)
Flag: **tjctf{n00b_h4x0r_b357_qu17}**
