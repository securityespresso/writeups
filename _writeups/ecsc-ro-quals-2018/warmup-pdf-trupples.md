---
title: Warmup PDF
contest: ECSC Romanian Quals 2018
authors: trupples
layout: writeup
---

The flag was written in the pdf with a completely transparent color and almost
outside the page boundaries. Opening the file with a web browser that supports
PDF files and reading the value of `document.body.innerText` revealed the flag:

`ECSC{7dd2c1f23fda4967b9496e773e6ee613092d450e41787985790da9df6e8116e2}`

Surrounded by two "decoy" flags:
`ECSC{tryhardertryhardertryhardertryhardertryhardertryhardertryharder!}`
