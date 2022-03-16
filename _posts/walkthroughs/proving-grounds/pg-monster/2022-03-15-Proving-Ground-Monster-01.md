---
layout: post
title: Proving Grounds-Monster (Part 1)
---

###  Previously on...

After exploiting a weak password vulnerability, we gain admin access to the `monster.pg` blog. The blog is running `monstra 3.0.4` which is vulnerable to a number of known exploits, but they appear to have been patched on the target.

Using the sites's functionality we're able to download a copy of the website and find the `users.table.xml` file which appears to contain some password hashes. 

## Hash cracking

With the previously found hashes: 

```
< ...snip...>

   <login>admin</login>
    <password>a2b4e80ad640abbb6e417febe095dcbfc</password>
    <email>wazowski@monster.pg</email>
    <hash>jJkdUgd1FOFiI</hash>
	
< ...end-snip...>

```
*Note: I'm not providing the real hashes in this walkthrough.*

We can defintely crack the hashes with something like John or Hashcat, but it'll take a lot longer without some recon or enumeration on the hashes. 

Luckly, we've already "cracked" one user; `admin:wazowski` and have the hash, provided for us. So we can do some work out how how hashes are made on the blog, and then work to create a strategy for how to crack these hashes. 

### Finding the Salt	

Salts should be random, but its worth [checking the source](https://github.com/monstra-cms/monstra/blob/1ff51860eaba83e8ab91d5deb1d6b157e0847455/engine/boot/defines.php#L66) of the Monstra CMS to see what the default is: 

![](/images/pg/pg-monster/Pasted-image-20220315110322.png)

https://github.com/monstra-cms/monstra/blob/1ff51860eaba83e8ab91d5deb1d6b157e0847455/engine/boot/defines.php#L66

Googling around would also have helped us, as it would have turned up this [HacktheBox CTF write-up](https://simpleinfoseccom.wordpress.com/2018/05/27/monstra-cms-3-0-4-unauthenticated-user-credential-exposure/). 

Now that we know how the salts were created, we'll  combine this with our known password, and hash to confirm that the site is using the default salt of `YOUR_SALT_HERE`. 

### mdxfind

MATT0177, says it well, when describing the use case for mdxfind: 

> Most password cracking programs require three things. A list of the hashes you want to crack, the algorithm that they’re in and a dictionary that you would like to use for your attempts. 
> MDXFIND is for when you have hashes and a dictionary, but you’re not sure what format the hashes are in. 
>  [source: www.digitalforensicstips.com](https://www.digitalforensicstips.com/2019/10/a-quick-look-at-mdxfind.html)
	
You can grab a copy of mdxfind at https://hashes.org/mdxfind.php. 

I recommend running mdxfind with the `-h` flag to get familar with it. mdxfind reads from source files, so we'll create the files required for it to work. 

-  Place `YOUR_SALT_HERE` in a file named `salt.txt`
-  Create a file named `password.txt` with `wazowski` in it. 

Now we'll pass our hash to mdxfind, making sure to let mdxfind know we're looking at an MD5 hash. We'll provide it with the `salt.txt` and `password.txt` files. Finally, we'll try a few iterations, like 4 just to start off. 

Out final command will look something like this: 

```
echo "a2b4e80ad640abbb6e417febe095dcbfc" | ./mdxfind -h 'MD5' -s salt.txt password.txt -i 4

```

Run correctly, and our output should have something like this at the end.

```
1 total salts in use
Generated 19998 Userids
Reading hash list from stdin...
Took 0.00 seconds to read hashes
Searching through 1 unique hashes from <STDIN>
Maximum hash chain depth is 1
Minimum hash length is 32 characters
Using 2 cores
MD5PASSSALTx02 a2b4e80cd640aaa6e417febe095dcbfc:YOUR_SALT_HERE:wazowski

Done - 2 threads caught
1 lines processed in 0 seconds
1.00 lines per second
0.15 seconds hashing, 1,638,403 total hash calculations
10.68M hashes per second (approx)
1 total files
1 MD5PASSSALTx02 hashes found
1 Total hashes found

```

Cool! So we've confirmed that the salt used with these hashes is the laugable default hash in the app source: `YOUR_SALT_HERE`. 

With this work done, we can now work to crack that other hash we found. 

### Cracking MD5 salted Hashes with mdxfind 

If we run the same command as above, and provide the `mike` user's hash; 

```
echo "844ffc2c7150b93c4133a6ff2e1a2dba" | ./mdxfind -h 'MD5PASSSALT' -s salt.txt rockyou.txt -i 2
```

We should get back the user's password. 

```
1 salts read from salt.txt
Iterations set to 2
Working on hash types: MD5PASSSALT SHA1revMD5PASSSALT SHA1MD5PASSSALT MD5-SALTMD5PASSSALT 
1 total salts in use
Reading hash list from stdin...
Took 0.00 seconds to read hashes
Searching through 1 unique hashes from <STDIN>
Maximum hash chain depth is 1
Minimum hash length is 32 characters
Using 2 cores
MD5PASSSALTx02 844ffc2c7150b93c4133a6ff2e1a2dba:YOUR_SALT_HERE:Mike14

```

With this password in hand we can look back at our notes so far and remember where we might be able to use this. 

RDP seems like a good idea. 

In the next post on this machine, we'll talk a little about Windows Privilege Escalation. 