---
layout: post
title: Proving Grounds: Monster (Part 0)
---


Monster is a cute litte box, that doesn't really have much of a *monster* angle, to it. Though, that's part of the charm, I think. Some good old enumeration will get you through to a shell. 

## Enumeration 

After running an initial nmap scan, (something as simple as `nmap -p- $target`) we get back a few list of some services running: http and https (80,443) and RDP (3389). 

### nmap

Taking a look at the http services running on the host, we can see that there's a lot of similarity between the the two services, so for now, we can assume they're the same and visit the http site. 

![[Pasted image 20220312135854.png]]

Before moving too far ahead, it was important for me log into the https site and take a look around. It's also a good idea to take a look at the SSL certificate the site issues and seek out subdomains that the cert may also be valid for. 

## Mike's Page 

![[Pasted image 20220312140512.png]]

The site belongs to Monster's Inc. character Mike Wazowski. 

Taking a look around, not much of the site appears functional. The resume download button, and other items don't link anywhere, but we note the use of a contact form at the bottom of the page. 

### GoBuster: Blog Discovery 

A GoBuster search, produces both  `/assets` and `/blog` directories. 

![[Pasted image 20220312140854.png]]

Navigating to the `/blog` page we find that the we're unable to proceed because we'll need to add `monster.pg` to our `/etc/hosts` file. 

After doing so we can visit the blog and begin our enumeration. 

## Blog

![[Pasted image 20220312141129.png]]

Just to have some enumeration running in the background, I start another GoBuster search on the `/blog` directory. 

The recommended way of doing website enumeration is going to be to "walk the site". Going page by and getting familar with how the site works and making a note of things that were modified, personalized, or would in any way be useful to us. 

Going through the header buttons and just scrolling down each page, taking a look at the source, and "getting a feel" for what the site does. 

### Vulnerability Discovery: Monstra 3.0.4

At the foot of the pages, we'll see that the site is running `monstra 3.0.4`. Using Searchsploit turns up a few solid exploits, but most require authentication. 

So we can see that we'll likely need to get on the site somehow, and currently, we don't have a need for brute forching users. 

### Vulnerability Discover:  Weak Passwords 

Listed under under `http://monster.pg/blog/users` we find some information about users. 

![[Pasted image 20220315104841.png]]

![[Pasted image 20220315104920.png]]

![[Pasted image 20220315104934.png]]

Having noted, this when we visit the login page, we'll have a few more combinations of users to try. 

Trying things like `admin:admin` are easy wins, and whenever we see a username, we should consider trying those too. While trying `mike:mike` fails, trying the username `admin` and the site owner's last name succeeds. 

With an authenticated user, and a known vulnerable version of software, this is the point where most folks want to jump ahead and get a shell. Searchploit turns up a bunch of RCE's and other interesting vectors. But none of these work. 

There's also the potential to upload or inject some php and get code execution on the web application. But this won't work either as these have been patched on this instance of Monstra. 

No, the easiest path is going to be to sit back and enumerate further. 

## Site Backup

![[Pasted image 20220315110322.png]]


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

```

And this is where we'll leave it for now, in the next post we'll discuss the best way to crack these hashes and get a shell on the box. 





# What is the site running? 

Its an apahce and php site, and the fonts are handled by popper, so not a mainstream CMS, that I can see. 

At a glace there's the is "meet me" thing writte in the corner, so I"ll note that for later: 

THere is entry for "meet me"  for searchsploit: 

```
Web-MeetMe 3.0.3 - 'play.php' Remote File Disclosure - php/webapps/4676.txt

```

While this is going I'll run a longer nmap scan in case we have more attack surface. 



Other findings: 

THere's a "download resume", but it doesn't actually work. nothing actually works.  


I'll start burp to capture traffic, since it might come in handy later. 

Robots,txt didn't have anything on it. 

## Gobuster 

gobuster dir -u http://$target -w /usr/share/wordlists/dirb/common.txt -t 20 -o gobuster-root.txt
nikto -h http://$target -o nikto.txt

Gobuster turns out a few 200 results w/ the shorter list, I'll start another just in case ... 

Looking for more content, I check the gobuster results and find /blog available. The site has "broken" css, but has a login page, I'm stopped here as I'll need to add "http://monster.pg/blog/admin" to my /etc/hosts file

I'll also add this host to my burp targets scope. 


I do and refresh the page


## Stopped ....


So this is the stupid part, because this machine runs great until I hit this part. So I'm gonna restart it and hope that it picks up a little better. But then I'm gonna say fuck it if it messes up again. 


## back ... 
 
/blog is built on something called Powered by Monstra 3.0.4

so we'll searchsploit that 

## Page Exploiration 

/blog 
/users
/login 
/registration 

Blog has some boring blog, but we might come back to it. 

/users has 2 users 

/login - 

doesn't work right way, with admin:admin, mike:mike; or monster:monster. 

/regstier

appears to be working so i'll create a jsullivan account w/ the pass123 pasword 

jsullivan:pass123
 
 monster@monster.pg

.... and I probably won't be able to register an account because, I can't solve the captcha. 

This concludes all of the pages, so now I'll restart burp and do the same stuff again. 

Revisiting the sites I can see that I'll have some info about the admin user, via this: 

http://monster.pg/blog/users/1

Username:	admin
Email:	wazowski@monster.pg
Registered:	22.2.2022

http://monster.pg/blog/users/2

Username:	mike
Email:	mike@monster.pg
Registered:	22.2.2022

 I'll try and see if I can find other users via the url .... but don't find anything. 


I try logging in again and this time take a look at the requests. 

The login stores a cookie that's got some kind of cooke that might prevent us from bruteforcing this login, which is kinda a relief. 


```
curl -i -s -k -X $'POST' \
    -H $'Host: monster.pg' -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0' -H $'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H $'Accept-Language: en-US,en;q=0.5' -H $'Accept-Encoding: gzip, deflate' -H $'Content-Type: application/x-www-form-urlencoded' -H $'Content-Length: 95' -H $'Origin: http://monster.pg' -H $'Connection: close' -H $'Referer: http://monster.pg/blog/users/login' -H $'Upgrade-Insecure-Requests: 1' \
    -b $'PHPSESSID=j5oj4ejd6r4r1vshfn0vojkcgj; _ga=GA1.2.1346206878.1647111407; _gid=GA1.2.236532325.1647111407; login_attempts=i%3A2%3B' \
    --data-binary $'csrf=1c9d1ef17709d8dfc8bd15a8b722d0bdc1dcf996&username=admi&password=nadmin&login_submit=Log+In' \
    $'http://monster.pg/blog/users/login'


```

So, now I'll go back up to either my lists, ports, or searchsploit: 

- Searchsploit Monstra 3.0.4

I get back a bunch of stuff via searchsploit but most are authnicated attacks. 

Or seem to be, I'll take a look at this: 

Monstra CMS 3.0.4 - (Authenticated) Arbitrary File Upload / Remote Code Execution| php/webapps/43348.txt
Monstra CMS 3.0.4 - Arbitrary Folder Deletion| php/webapps/44512.txt

and then the XSS vulns that exist ... I also don't have any enumeration running, so I'll try and re-run that nmap scan while I wait. 

I'll need to be authenticated as a user for any of these attacks to work ... 

Theres an XSS bug, that seems kinda cool, but I'll still need to be authenticated. 

Which again is leading me to think I might have to brute force the http form ... but let's keep on waiting for that one. 


## Dead end  ...

I'll look at my scans and report back, but now I have something to look for again. 

- there's stlll the ssl site 
- nikto doens't turn up much
- there's also RDP, which can be bruteforced? but maybe not. nmap all port doesn't turn up too much more
- I'll take a look at the ssl cert for more info: 

!! - After looking at my gobuster results, I'll make to take a look at LFI and other things that might be possible. 
!! = I'm stopping my gobuster scan because I think its killing hte box somehow

.... and restarting it again, and take a look at teh following when they come back: 

/blog                 (Status: 301) [Size: 342] [--> http://192.168.175.180/blog/]
/assets               (Status: 301) [Size: 344] [--> http://192.168.175.180/assets/]
/Blog                 (Status: 301) [Size: 342] [--> http://192.168.175.180/Blog/]
/Assets               (Status: 301) [Size: 344] [--> http://192.168.175.180/Assets/]


...since these came back, I'll take a look at LFI's 

## Longer Commands: 

sudo nmap -sS -sV -sC --min-rate=10000 -p- -oN fast-nmap -vv $target; wait; sudo nmap -sS -sV -sC  -p- -oN nmap-ALL-ports -vv $target

and nothing, but 403s ... so now I think I'll do searhsploit for earlier versions and re lookat some earlier scans. 


## SQLi? 

I was never too sure on what version fo montra we were running so I tried an earlier version and found this sqli: 



... and here I realize I haven't run the same gobuster scans on /blog, so I'll do that now: 

 
searchsploit -m php/webapps/38769.txt

This sqli doesn't seem to immeidately work. so I PANICCKKKKKKK and look at the hint .... 

somethign about the admin page --- which I've found ... 

and the credentials are 

admin:wazowski

the username and last name :/. 

so now I'll try and find a version and cirlcle back to all the yummpers exploits. 

and if iwasn't a doofus, I'd remmber that the info could be found  in the footer of the /blog page. 

I look at the arbitrary file upload, (php/webapps/48479.txt)  ... and I'll try it first. 

The text says that I'll have to upload w/ the file extension in CAPS, but I'll try without it first. 

These don't work off the bat, so I'll tr somethign else, like edit a post to have ethe content of the malicious .php file 

and that doesn't work either, but i'll take a look at other php file extensions,


and then the other exploits: 


I was hoping that the XSS would work, I even got it to work sort of, but I couldn't get the correct CSRF token, so it didn't work and forget to do what I should done to begin wtih .... look at what I'm able to do as the use. 

# Creating a back up of hte site

I visit the backup site, and find that I should be able to 



















This is the correct command to run: 


echo "844ffc2c7150b93c4133a6ff2e1a2dba" | ./mdxfind -h 'MD5PASSSALT' -s salt.txt /usr/sharewordlists/rockyou.txt -i 2
