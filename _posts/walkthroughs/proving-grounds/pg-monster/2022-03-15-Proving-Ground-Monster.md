---
layout: post
title: Proving Grounds-Monster (Part 0)
tags: proving_grounds, oscp
---


Monster is a cute litte box, that doesn't really have much of a *monster* angle, to it. Though, that's part of the charm, I think. Some good old enumeration will get you through to a shell. 

## Enumeration 

After running an initial `nmap` scan, (something as simple as `nmap -p- $target`) we get back a few list of some services running: http and https (`80,443`) and RDP (`3389`). 

### nmap

Taking a look at the http services running on the host, we can see there are a lot of similarities between the the two services, so for now, we can assume they're the same and visit the http site. 


![nmap scan info](/images/pg/pg-monster/Pasted-image-20220312135854.png)



Before moving too far ahead, it was important for me log into the https site and take a look around. It's also a good idea to take a look at the SSL certificate the site issues and seek out subdomains that the cert may also be valid for. 

Neither turns up anything, but gives me a good feeling knowing I checked. 

## Mike's Page 

![mike's super cool site](/images/pg/pg-monster/Pasted-image-20220312140512.png)

The site belongs to Monster's Inc. character Mike Wazowski. 

Taking a look around, not much of the site appears functional. The resume download button, and other items don't link anywhere, but we note the use of a contact form at the bottom of the page. 

### GoBuster: Blog Discovery 

A GoBuster search, produces both  `/assets` and `/blog` directories. 

![](/images/pg/pg-monster/Pasted-image-20220312140854.png)

Navigating to the `/blog` page we find that the we're unable to proceed because we'll need to add `monster.pg` to our `/etc/hosts` file. 

After doing so we can visit the blog and begin our enumeration. 

## Blog

![](/images/pg/pg-monster/Pasted-image-20220312141129.png)

Just to have some enumeration running in the background, I start another GoBuster search on the `/blog` directory. 

The recommended way of doing website enumeration is going to be to "walk the site". Going page by page and getting familar with how the site works and making a note of things that were modified, personalized, or would in any way be useful to us. 

Going through the header buttons and just scrolling down each page, taking a look at the source, and "getting a feel" for what the site does. 

### Vulnerability Discovery: Monstra 3.0.4

At the foot of the pages, we'll see that the site is running `monstra 3.0.4`. Using searchsploit, we turn up a few exploits, but most require authentication. 

We won't get attached to it, but we can now start to forumlate a hypothesis which states we'll need to impersonatea user to proceed further into the target. Currently, we don't have any usernames, so we'll continue to walk the site. 

### Vulnerability Discover:  Weak Passwords 

Listed under under `http://monster.pg/blog/users` we find some information about users. 

![](/images/pg/pg-monster/Pasted-image-20220315104841.png)

![](/images/pg/pg-monster/Pasted-image-20220315104920.png)

![](/images/pg/pg-monster/Pasted-image-20220315104934.png)

This is great! And we'll note it down, as these will be the key to cracking the login problem.

### Login Page

Trying things like `admin:admin` are easy wins, and whenever we see a username, we should consider trying those too. While trying `mike:mike` fails, trying the username `admin` and the site owner's last name succeeds. 

With an authenticated user, and a known vulnerable version of software, this is the point where most folks want to jump ahead and get a shell. Searchploit turns up a bunch of RCE's and other interesting vectors. But none of these work. 

With known exploits failing us, we might want to upload or inject some php and get code execution on the web application. But this won't work either as these have been patched on this instance of Monstra. 

No, the easiest path is going to be to sit back and enumerate further. 

## Abusing Site Features/Functionality: Site Backup

![](/images/pg/pg-monster/Pasted-image-20220315110322.png)


Walking the application, we'll find that we have the ability to download a back up of the site. We'll create a backup, download it and unzip it and find that there are *a lot* of files here. 

A lot of the files seem to be `.html` files and images, but since we're interested in the back end, we'll take a look at the `.xml` files. 

```
-rw-r--r-- 1 kali kali  458 Mar 15 14:06 blog.manifest.xml
-rw-r--r-- 1 kali kali  471 Mar 15 14:06 captcha.manifest.xml
-rw-r--r-- 1 kali kali  534 Mar 15 14:06 codemirror.manifest.xml
-rw-r--r-- 1 kali kali  490 Mar 15 14:06 markdown.manifest.xml
-rw-r--r-- 1 kali kali  489 Mar 15 14:06 markitup.manifest.xml
-rw-r--r-- 1 kali kali  507 Mar 15 14:06 menu.table.xml
-rw-r--r-- 1 kali kali 1.9K Mar 15 14:06 options.table.xml
-rw-r--r-- 1 kali kali 1.7K Mar 15 14:06 pages.table.xml
-rw-r--r-- 1 kali kali 3.6K Mar 15 14:06 plugins.table.xml
-rw-r--r-- 1 kali kali  470 Mar 15 14:06 sandbox.manifest.xml
-rw-r--r-- 1 kali kali  820 Mar 15 14:06 users.table.xml

```

The file `users.table.xml` is the most interesting one here, and grepping the file for the string `password` 

We find some interesting hashes. 

`grep -i 'password' users.table.xml --color=auto`

```
<..snip..>

\<root><options><autoincrement>2</autoincrement></options><fields><login/><password/><email/><role/><date_registered/><firstname/><lastname/><login/><twitter/><skype/><hash/><about_me/></fields><users><id>1</id><uid>de58425259</uid><firstname/><lastname/><twitter/><skype/><about_me/><login>admin</login><password>a2b4e80cd640aaa6e417febe095dcbfc</password><email>wazowski@monster.pg</email><hash>jJkdUX1FOFiI</hash><date_registered>1645512776</date_registered><role>admin</role></users><users><id>2</id><uid>800c7d9797</uid><firstname/><lastname/><twitter/><skype/><about_me/><login>mike</login><password>844ffc2c7150b93c4133a6ff2e1a2dba</password><email>mike@monster.pg</email><hash>8vPjvUPDHhRp</hash><date_registered>1645512909</date_registered><role>user</role></users></root>

<...end-snip...>

```

And this is where we'll leave it for now, in the next post we'll discuss the best way to crack these hashes and get a shell on the box. 



