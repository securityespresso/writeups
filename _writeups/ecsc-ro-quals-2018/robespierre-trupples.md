---
title: Robespierre
contest: ECSC Romanian Quals 2018
authors: trupples
layout: writeup
---

This python script does the number splitting and adding in a very space and
time inefficient manner but it works well as 24947 is a relatively small
number.

```py
s = " ".join([str(x) for x in range(1, 24948)]).replace("0", " ").split(" ")
sum = 0
print(s)
for k in s:
  if k != '':	# Some elements are '' because of double zeroes
	  sum += int(k)

print (sum) # => 208464993
```

The flag is `ECSC{208464993}`
