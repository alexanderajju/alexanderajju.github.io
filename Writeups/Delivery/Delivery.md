---
permalink: /delivery
layout: post
author: Ippsec
title: Delivery
date: 2021-05-22
publish: True

description: "Ready is a medium difficulty Linux machine. A vulnerable version of GitLab server leads to a remotecommand execution, by exploiting a combination of SSRF and CRLF vulnerabilities. Bad permission on abacked up configuration file of the Gitlab server, reveals a password that is found to be reusable for theuser root, inside a docker container. After root access is acquired, escaping the container is possible sinceit is running in privileged mode."
---

# Skills Learned

- Email impersonation
- Intermediate password cracking

# Enumeration

## Nmap

- port 22
- port 80

```bash
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-19 20:40 IST
Nmap scan report for 10.10.10.222
Host is up (0.20s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey:
|   2048 9c:40:fa:85:9b:01:ac:ac:0e:bc:0c:19:51:8a:ee:27 (RSA)
|   256 5a:0c:c0:3b:9b:76:55:2e:6e:c4:f4:b9:5d:76:17:09 (ECDSA)
|_  256 b7:9d:f7:48:9d:a2:f2:76:30:fd:42:d3:35:3a:80:8c (ED25519)
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Welcome
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.91%E=4%D=5/19%OT=22%CT=1%CU=37261%PV=Y%DS=2%DC=T%G=Y%TM=60A52AB
OS:B%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=106%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST1
OS:1NW7%O6=M54DST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS:(R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   205.46 ms 10.10.14.1
2   205.55 ms 10.10.10.222

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 70.59 seconds

```

## Port 80

![](/Writeups/Delivery/Pasted image 20210519204541.png)

On exploring the source list or Bruteforcing the subdirectories we couldn't find anything. Therefore looking more towards the `contact` section where we can find `help desk`and `mattermost server`.

![](/Writeups/Delivery/Pasted image 20210519204944.png)

![](/Writeups/Delivery/Pasted image 20210519204616.png)

On visting the help desk we need to add hostname to hostfile.

![](/Writeups/Delivery/Pasted image 20210519204734.png)

## Support Center

![](/Writeups/Delivery/Pasted image 20210519204821.png)

## Mattermost

![](/Writeups/Delivery/Pasted image 20210519205100.png)

Mattermost is an open-source, self-hostable online chat service with file sharing, search, and integrations. It is designed as an internal chat for organisations and companies, and mostly markets itself as an open-source alternative to Slack and Microsoft Teams.

# Foothold

- Sign up for new user

![](/Writeups/Delivery/Pasted image 20210519205229.png)

![](/Writeups/Delivery/Pasted image 20210519205553.png)

Signup almost screwed up Verification required for mail provided for signup.
Without knowing the code, or a valid email, there’s no way to bypass the email verification. Machines on HTB won’t send emails out .Looks like we’ve hit a dead-end. Time to moveon and investigate the Help Desk service.

![](/Writeups/Delivery/Pasted image 20210519205535.png)

- In support center there is a way to open a ticket

- Creating a ticket using username and email

![](/Writeups/Delivery/Pasted image 20210519205706.png)

After submitting ticket we got ticket id and email .

![](/Writeups/Delivery/Pasted image 20210519205738.png)

3175892@delivery.htb(my 1st section died)
8757468@delivery.htb(created new one)

- With the new email signup for mattermost server.

![](/Writeups/Delivery/Pasted image 20210519210621.png)

- For the verification after signup process , Check the status of the ticket in mattermost sever using the ticket id and email used for creating the new ticket

![](/Writeups/Delivery/Pasted image 20210519210840.png)

- For the verification, use the verification link provided in the status page.

http://delivery.htb:8065/do_verify?token=aid6trt41zfhchtoeydz1sww39wyw3gkhcu381g8kkxr7fmi6f8mb9h4cd8mb9h4cd8mdnzrt&email=8757468%40delivery.htb

## Verification Sucess

- Login using our creds

![](/Writeups/Delivery/Pasted image 20210519211912.png)

message to login the mailserver with password

| user          | password        |
| ------------- | --------------- |
| maildeliverer | Youve_G0t_Mail! |

![](/Writeups/Delivery/Pasted image 20210519212203.png)

![](/Writeups/Delivery/Pasted image 20210519212525.png)

- ssh in using creds above

## User flag

![](/Writeups/Delivery/Pasted image 20210519212632.png)

# Privilege Escalation

```bash
maildeliverer@Delivery:/opt/mattermost/config$ ls
README.md  cloud_defaults.json  config.json
```

- Reading the config.json in mattermost config we got Username and password to the MariaDB

![](/Writeups/Delivery/Pasted image 20210519212851.png)

| user   | password              |
| ------ | --------------------- |
| mmuser | Crack_The_MM_Admin_PW |

```bash
maildeliverer@Delivery:/opt/mattermost/config$ mysql -u mmuser -D mattermost -p
Enter password:
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 93
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [mattermost]>

```

- Use Mattermost database
- select username and password and email from users table

```bash
MariaDB [mattermost]> SELECT Username,Password,Email FROM Users;
+----------------------------------+--------------------------------------------------------------+-------------------------+
| Username                         | Password                                                     | Email                   |
+----------------------------------+--------------------------------------------------------------+-------------------------+
| surveybot                        |                                                              | surveybot@localhost     |
| c3ecacacc7b94f909d04dbfd308a9b93 | $2a$10$u5815SIBe2Fq1FZlv9S8I.VjU3zeSPBrIEg9wvpiLaS7ImuiItEiK | 4120849@delivery.htb    |
| 5b785171bfb34762a933e127630c4860 | $2a$10$3m0quqyvCE8Z/R1gFcCOWO6tEj6FtqtBn8fRAXQXmaKmg.HDGpS/G | 7466068@delivery.htb    |
| root                             | $2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO | root@delivery.htb       |
| ff0a21fc6fc2488195e16ea854c963ee | $2a$10$RnJsISTLc9W3iUcUggl1KOG9vqADED24CQcQ8zvUm1Ir9pxS.Pduq | 9122359@delivery.htb    |
| channelexport                    |                                                              | channelexport@localhost |
| 9ecfb4be145d47fda0724f697f35ffaf | $2a$10$s.cLPSjAVgawGOJwB7vrqenPg2lrDtOECRtjwWahOzHfq1CoFyFqm | 5056505@delivery.htb    |
+----------------------------------+--------------------------------------------------------------+-------------------------+

```

we found hash for the root user

| user | hash                                                         |
| ---- | ------------------------------------------------------------ |
| root | $2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO |

Cracking the hash of the root user

_Also please create a program to help us stop re-using the same passwords everywhere…. Especially those that are a variant of “PleaseSubscribe!”_ comment that was made on the MatterMost server.

- In order to crack the hash we need to create a wordlist using [hashcat rules](https://github.com/praetorian-inc/Hob0Rules.git)

```bash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Warning: Only 2 candidates left, minimum 6 needed for performance.
PleaseSubscribe!21 (root)
Use the "--show" option to display all of the cracked passwords reliably
Session completed

```

| user | password           |
| ---- | ------------------ |
| root | PleaseSubscribe!21 |

## Root

```bash

maildeliverer@Delivery:/opt/mattermost/config$ su
Password:
root@Delivery:/opt/mattermost/config# cd
root@Delivery:~# ls
mail.sh  note.txt  py-smtp.py  root.txt
root@Delivery:~#

```

[back](/Writeups/writup)
