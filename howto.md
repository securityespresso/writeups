---
title: "How-To"
layout: default
---

# How-To

Hello!

If you want to contribute to the `jmp 0xc0ffee` writeup collection
you can do so by following these quick steps:

## 1. Fork this repo on Github

The first thing you'll have to do is fork
<a href="https://github.com/securityespresso">securityespresso</a>/<a href="https://github.com/securityespresso/writeups">writeups</a>
so that you'll have an identical repo that you're allowed to modify.

## 2. Add yourself to the author list

After that you should add yourself to the author list by editing `_data/authors.yml` and
adding an entry such as the example one below. Keep in mind that the first handle will be
the one you will have to use to claim your writeups. You can skip the email field if you
wish, we just need one handle for everything to work ok.

```yml
... other authors ...

- email: green@orange.com
  handles:
    - GreenOrange
    - gr33n0r4ng3
```

## 3. Add your writeup

Considering you've already written your writeup in Github-Flavored Markdown, you just
need to add some metadata at the beginning of the file so that Jekyll knows how to handle
it. Here's an example:

```
---
title: YAPS
contest: TimCTF Finals 2018
authors: trupples
layout: default
---

... writeup contents ...
```

Change the title, contest and authors fields as necessary. If multiple people worked on
this writeup, set the authors field to everyone's handles separated by commas.

Now just place the writeup in the `_writeups` folder.

## 4. Commit, push, pull request

Awesome, you're almost there! The last step is pushing the changes to your fork of the
repo and then submitting a Pull Request. We'll be sure to take a look and check that
it's all ok as quickly as possible.
