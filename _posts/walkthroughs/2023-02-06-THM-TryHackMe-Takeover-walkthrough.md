---
layout: post  
title: TryHackMe Walkthrough Takeover 
tags: ffuf, apache, subdomain fuzzing, TLS, SSL 
---

Takeover is an easy rated box on the TryHackMe platform. The challenge was created by some THM all-stars: [JohnHammond](https://tryhackme.com/p/JohnHammond), [cmnatic](https://tryhackme.com/p/cmnatic), [fumenoid](https://tryhackme.com/p/fumenoid) and [timtaylor](https://tryhackme.com/p/timtaylor). 

You can play Takeover by visiting the following link: https://tryhackme.com/room/takeover

## Background 

The description reminds us to add the machine IP and domain target to our `/etc/hosts` file. 

We can do this by opening the our `/etc/hosts` file in vim. We'll need to use sudo for it.  

```bash
sudo vim /etc/

```

we'll add the following line: 

```bash 

10.10.X.X futurevera.thm

```

And this is essentially 80% of the path to the flag on this challenge. 

## Enumeration 

The hint provided to tells us this is a sub-domain challenge. Even still, we should always run an nmap scan, as well as any other enumeration we might like to run. 

### Viewing the Website - Approving Unknown SSL Certificate

When we enter the target domain into our web browser `http://futurevera.thm` we're redirected to an **https** version of the site. 

We're warned that the certificate isn't secure. On a CTF and **always** -- we should view the certificate. Doing so in this case doesn't provide us any new information about the host, but its a good practice to get into. 

Eventually, after checking the source of the page, we'll want to start the process of fuzzing for subdomains. 

### Fuzzing with ffuf 

It's generally a good idea to use a variety of tools for the same tasks. So the following is only how I performed this fuzzing. The same could be done with `wfuzz`, or `gobuster`.  These tools fall in and out of fashion and in some part come down to personal preference. 

I'm going to use `ffuf`. Which you is pre-installed in Kali and ParrotOS, and can also be acquired here: https://github.com/ffuf/ffuf

#### Fuzzing Methodology 

Since I'm using ffuf, we'll start off by issuing a curl command.

```bash 

# request
curl -I futurevera.thm

# response

HTTP/1.1 302 Found
Date: Mon, 06 Feb 2023 20:15:46 GMT
Server: Apache/2.4.41 (Ubuntu)
Location: https://futurevera.thm/
Content-Type: text/html; charset=UTF-8


```

We'll note the `Apache/2.4.41` server information. We can use this to assume, the site is using Apache's virtual host option. We can read more about this here: https://httpd.apache.org/docs/2.4/vhosts/

With this information we can start our host name fuzzing: 

```bash 

ffuf -ic -c -u https://10.10.X.X/ -w $dbust -H 'Host: FUZZ.futurevera.thm' -t 20 

```

**Flag explanation**

- `ic` - ignore comments in the wordlist 
- `c`  - colorize our output
- `u` - url to fuzz
- `w` - provide our wordlist. Here the `directory-list-2.3-medium.txt` is aliased to the `$dbust` variable
- `H` - the contents of our host header, more on this below
- `t` - number of threads to use

There are two parameters in the above command that might trip us up: 

1) It's important to add append the IP address we provided in the url parameter with a slash (`/`) 
2) Since we're fuzzing the Host header, its also important that we place a space between the http host header, and the url we're fuzzing. 

The above fuzz will give a large output, and we'll need to utilize a filter. In `fuff` we'lll use something simple like fuzzing on the size of the response. Our new command is the following: 

```bash

ffuf -ic -c -u https://10.10.X.2X/ -w $dbust -H 'Host: FUZZ.futurevera.thm' -t 20 -fs 4605

```

And we'll get back the following response: 

```bash

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.5.0 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : https://10.10.X.X/
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Header           : Host: FUZZ.futurevera.thm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 20
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 4605
________________________________________________

blog                    [Status: 200, Size: 3838, Words: 1326, Lines: 81, Duration: 165ms]
support                 [Status: 200, Size: 1522, Words: 367, Lines: 34, Duration: 144ms]

```

We get back some new end-points! 

Its worth nothing that the list I'm using is relatively large and contains a lot of useful words for CTFs. The endpoints discovered are: 

- blog 
- support 

## Further Enumeration 

If we try and visit any of the endpoints we've found, we'll get an error letting us know the site can't be found. 

We'll need to add the new endpoints to our `/etc/hosts` file, like we did above. Same sudo command, and either on its own line or added to the same line as our other hostname, we'll add the following: 

```bash

10.10.X.X futurevera.thm blog.futurevera.thm support.futurevera.thm

```

Visiting these sites will ask us, once again to approve the certificates. **And just like before**, we'll make sure to inpect the certificates. 

The `blog.futurevera.thm` SSL certificate doesn't provide us any new information,  but the `support.futurevera.thm` certificate provides us another domain for us. 

## Flag Retrieval 

Adding the new-found domain to our `/etc/hosts` file will get us a little further towards reading teh flag. 

Unfortunately,  this secret domain appears to be the same or similar content we've seen before. 

A simple form of enumerating this site would include visiting the NON-https site to view any content. 

```bash 
#request 
curl -I -k http://[REDACTED].support.futurevera.thm 


# response
HTTP/1.1 302 Found
Date: Mon, 06 Feb 2023 20:38:13 GMT
Server: Apache/2.4.41 (Ubuntu)
Location: http://flag{flagcontents!!!!}.s3-website-us-west-3.amazonaws.com/
Content-Type: text/html; charset=UTF-8

```


And we can read the flag! 

## Thanks! 

If this post was helpful to you, please feel free to reach out, or consider buying me a coffee.

Thanks - happy hacking!

[Buy me a coffee](http://buymeacoffee.com/displayerror)
