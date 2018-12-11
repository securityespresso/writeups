---
title: ReCurse
contest: OtterCTF 2018
authors: Lucian Nitescu
layout: writeup
---

### Description:

Found this nested zip in Morty's PC. what is it that he is hiding?

[Download](https://nitesculucian.github.io/uploads/otter1/a.zip)

### Stats:

150 points / 94 solvers

### Solution:  

The challenge started with a ```.zip``` file which contained multiple zipped files within other zipped files as you can see in the following example:

![](https://nitesculucian.github.io/uploads/otter1/image6.png)

My approach was rather brute: I unzipped one file in a folder and within the newly created folder, I repeated my actions. Here is the single bash command that I executed:

```bash
while true; do unzip $(ls \*.zip) -d $(ls \*.zip). && cd $(ls \*.zip).; done
```

Output:

![](https://nitesculucian.github.io/uploads/otter1/image2.png)

The resulting working directory and the retrieved files:

![](https://nitesculucian.github.io/uploads/otter1/image8.png)

```w.zip``` is the last zip archive within the chain and requires a password to extract the archived text file.

![](https://nitesculucian.github.io/uploads/otter1/image5.png)

From the working directory path I decided to strip all the extension names (```.zip```) and other unnecessary file names:

```
/home/nli/Desktop/otterctf/ReCurse/a.zip./H.zip./R.zip./0.zip./c.zip./H.zip./M.zip./6.zip./L.zip./y.zip./9.zip./3.zip./d.zip./3.zip./c.zip./u.zip./Z.zip./X.zip./h.zip./v.zip./d.zip./G.zip./l.zip./j.zip./Y.zip./W.zip./5.zip./p.zip./b.zip./W.zip./F.zip./s.zip./c.zip./2.zip./Z.zip./v.zip./c.zip./n.zip./N.zip./h.zip./b.zip./G.zip./U.zip./u.zip./b.zip./m.zip./V.zip./0.zip./L.zip./3.zip./N.zip./h.zip./b.zip./G.zip./U.zip./v.zip./M.zip./z.zip./k.zip./z.zip./N.zip./T.zip./M.zip./t.zip./M.zip./i.zip./1.zip./m.zip./Z.zip./W.zip./1.zip./h.zip./b.zip./G.zip./U.zip./t.zip./c.zip./2.zip./1.zip./h.zip./b.zip./G.zip./w.zip./t.zip./Y.zip./2.zip./x.zip./h.zip./d.zip./y.zip./1.zip./B.zip./c.zip./2.zip./l.zip./h.zip./b.zip./i.zip./1.zip./v.zip./d.zip./H.zip./R.zip./l.zip./c.zip./n.zip./M.zip./u.zip./Y.zip./X.zip./N.zip.
```

Output:

```
aHR0cHM6Ly93d3cuZXhvdGljYW5pbWFsc2ZvcnNhbGUubmV0L3NhbGUvMzkzNTMtMi1mZW1hbGUtc21hbGwtY2xhdy1Bc2lhbi1vdHRlcnMuYXN
```

After I decoded the above base64 string, I obtained the following link:

[https://www.exoticanimalsforsale.net/sale/39353-2-female-small-claw-Asian-otters.as](https://www.google.com/url?q=https://www.exoticanimalsforsale.net/sale/39353-2-female-small-claw-Asian-otters.as&sa=D&ust=1544480771866000)

I had to add the ```p``` letter to the end of the link in order to access the page:

[https://www.exoticanimalsforsale.net/sale/39353-2-female-small-claw-Asian-otters.asp](https://www.google.com/url?q=https://www.exoticanimalsforsale.net/sale/39353-2-female-small-claw-Asian-otters.asp&sa=D&ust=1544480771867000)

![](https://nitesculucian.github.io/uploads/otter1/image9.png)

By clicking on the ```User Review``` link, I was redirected to [http://www.birple.com/users.asp?id=Brking1991@gmail.com&sid=175](https://www.google.com/url?q=http://www.birple.com/users.asp?id%3DBrking1991@gmail.com%26sid%3D175&sa=D&ust=1544480771868000) website and page. At first, I thought that this was a dead end, but after multiple tries and failures I decided to use the leaked email (Brking1991@gmail.com) as the password for my last archive file:

![](https://nitesculucian.github.io/uploads/otter1/image3.png)

Output:

![](https://nitesculucian.github.io/uploads/otter1/image1.png)

Obtaining the flag:

![](https://nitesculucian.github.io/uploads/otter1/image4.png)

```
flag{Recursion_1S_T3rribl3_AnD_1_H4t3_My_L1F3!!}
```

![](https://nitesculucian.github.io/uploads/otter1/image7.png)