---
layout: post
title: Linux Forensics Cheatsheet
tags: linux, forensics, cheatsheet, incident response
---

Recently, I finished up the [LInux Forensics Room](https://tryhackme.com/room/linuxforensics) on TryHackMe and found a lot of really great refreshers on concepts I think are relevant for Penetration Testers, CTF Players and wannabe Red Teamers. 

Here's my LInux Forensics cheatsheet, its also available on Github. 

##  OS and account information

Getting release information: 

```
cat  /etc/os-release
```

## Finding User Accounts: 

The passwd is usually world readable by default and can be used to enumerate other users on the machine. 

```
cat /etc/passwd
```

We can clean up the output  w/ the following: 

```
cat /etc/passwd | column -t -s :
```

## Group Information
We can get information about groups in the following way: 
`/etc/group`

Example: 

```
user@machine$ cat /etc/group 
root:x:0: 
daemon:x:1: 
bin:x:2: 
sys:x:3: 
adm:x:4:syslog,ubuntu 
tty:x:5:syslog
```

Here' we can see the user `adm` belongs to the `syslog` and `ubuntu` groups. 

The `x` signifies that the user has a password stored in the /etc/shadow file. 

## Sudoers List 

We can view the sudoers list, or users allowed to upgrade their privileges by viewing. 
`/etc/sudoers`

## Login information

Found in the `/var/log`, we can view log files. These include: 

```
- wtmp
- btmp
```

These contain information about failed logins. `wtmp` keeps historical data about logins. These files are binary files and can be viewed with the `last` command. 


## Authentication logs

All authenticagted users are logged in the authlog. These can be found at:

`/var/log/auth.log`

You'll need to be root or allowed to view these files. 

**Example usage:** 

```
cat /var/log/auth.log | tail
```

## Hostname

**Example:**

`cat /etc/hostname`

## Timezone

`cat /etc/timezone`

## Network Configuration 

`/etc/network/interfaces`

## Active network connections

We primarly will use system tools like `netstat`

```
netstat -pant
```

## Running processes
`ps aux`

## DNS information

Files like `/etc/hosts` contain configuration information for DNS assignments. 

`cat /etc/hosts`

Information about DNS resolvers (how linux hosts talks to DNSServers) can be found in `/etc/resolv.conf`

`cat /etc/resolv.conf`

* * * * 

## Persistence Mechanisms

## Cron jobs

```
cat /etc/crontab
```

## Service startup

```
cat /etc/init.d
```


## .Bashrc

When a bash shell is started it runs commands through the `.bashrc` file which can be  found in the users home directory. 
/var/
`cat ~/.bashrc`

## Sudo execution history

All the commands that are run on a Linux host using `sudo` are stored in the auth log. We already learned about the auth log in Task 3. We can use the `grep` utility to filter out only the required information from the auth log.

```
user@machine$ cat /var/log/auth.log* |grep -i COMMAND|tail

```

## Bash history

Any commands other than the ones run using `sudo` are stored in the bash history. Every user's bash history is stored separately in that user's home folder. Therefore, when examining bash history, we need to get the bash_history file from each user's home directory. It is important to examine the bash history from the root user as well, to make note of all the commands run using the root user as well.

```
user@machine$ cat ~/.bash_history
```

## Files accessed using vim

Vim keeps logs. So we can and should access these: 

```
cat ~/.viminfo

```

# Log FIles 
Log files are insanely important for forensics investigations. 

Log files can be found at: `/var/log`

## Syslog 
The Syslog contains messages that are recorded by the host about system activity. The  detail is configurable through the logging level. 

We can use `cat, head, more`, and `less.


**Example**

```
cat /var/log/syslog* | head
```

## Auth logs

```
cat /var/log/auth.log* |head
```

## Third Party Logs 

Similar to `sys` and `auth` logs we can find other types of logs in `/var/log/`. 