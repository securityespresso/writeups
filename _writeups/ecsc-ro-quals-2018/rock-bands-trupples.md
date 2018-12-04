---
title: rock-bands
contest: ECSC Romanian Quals 2018
authors: trupples
layout: writeup
---

The attached archive contains an USB key image with NTFS. The file we're most
interested in is an [Alternate Data Stream](https://support.microsoft.com/en-us/help/105763/how-to-use-ntfs-alternate-data-streams)
which makes mounting the image or using 7zip, photorec or binwalk to extract
the files either not work at all or only extract the other, irrelevant files.

[Autopsy](http://www.sleuthkit.org/autopsy/) worked, though, and we can use it
to extract `vlc-3.0.3-win32.exe:rockbands.exe` (the colon indicates that it's
an ADS). Running the extracted executable (which is a self extracting archive)
created a pcap file named `InternalNetwork.pcap`.

A brief look through the packets was enough to spot the flag which was encoded
in base64:
`RUNTQ3tkZjMwNDEyOGVkMGFlMmJlMTFjOTg5NmViNzk1NmVmOWQ4Yjc3ZDM1ZGNjNjkxYjdmMDIyNmU0NDNjOTc5MTQyfQ==` => `ECSC{df304128ed0ae2be11c9896eb7956ef9d8b77d35dcc691b7f0226e443c979142}`
