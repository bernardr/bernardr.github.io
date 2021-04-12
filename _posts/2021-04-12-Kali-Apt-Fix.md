---
layout: post
title: FIXED: Kali 2020.2 OVA VIRTUALBOX - HASH SUM MISMATCH
---

This fixes an issue with apt in Kali. 

```
$ sudo bash

$ mkdir /etc/gcrypt
$ echo all >> /etc/gcrypt/hwf.deny
$ sudo apt-get update
```

[Source](https://forums.kali.org/showthread.php?48822-Kali-2020-2-OVA-VIRTUALBOX-HASH-SUM-MISMATCH)

There's a good explanation of how this works [here](https://askubuntu.com/questions/1235914/hash-sum-mismatch-error-due-to-identical-sha1-and-md5-but-different-sha256)


