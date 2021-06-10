---
permalink: /scriptkiddie
layout: post
author: 0xdf
title: Scriptkiddie
date: 2021-06-05
publish: True

description: "ScriptKiddie is an easy difficulty Linux machine that presents a Msf venom apk template injection, along with classic attacks such as OS command injection and an insecure passwordless sudo configuration. Initial foothold on the machine is gained by uploading a malicious .apk file from a web interface that calls a vulnerable version of msfvenom to generate downloadable payloads. Once shell is obtained, lateral movement to a second user is performed by injecting commands into a log file which provides unsanitized input to a Bash script that is triggered on file modification. This user is allowed to run msfconsole as root via sudo without supplying a password, resulting in the escalation of privileges."
---

# Skills Learned

- OS Command Injection in command arguments
- Running system commands from Metasploit console

# Enumeration

## Nmap
* Port 5000
* Port 22

```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)
|   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)
|_  256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)
5000/tcp open  http    Werkzeug httpd 0.16.1 (Python 3.8.5)
Device type: firewall
Running: Fortinet embedded
OS CPE: cpe:/h:fortinet:fortigate_100d
OS details: Fortinet FortiGate 100D firewall
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8080/tcp)
HOP RTT    ADDRESS
```

Port 5000

![](/Writeups/scriptkiddie/Pasted image 20210609191517.png)

The website hosted contained nmap for scanning 100 ports, Payloads using msf venom and search exploit;
 
 On further exploration we found a apk templete injection 
 
 ```bash
 ┌[Parrot]─[/home/aju/HTB/scriptkiddie]
└╼aju$searchsploit msfvenom
------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                     |  Path
------------------------------------------------------------------------------------------------------------------- ---------------------------------
Metasploit Framework 6.0.11 - msfvenom APK template command injection                                              | multiple/local/49491.py
------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

# Foothold

using metasploit for further exploitataion

![](Pasted image 20210609193509.png)

set LHOST to tun0 and run the exploit

![](Pasted image 20210609193752.png)

after running exploit an apk will be created home directory
`msf.apk stored at /home/aju/.msf4/local/msf.apk`

upload the apk template to port 5000 website 

![](Pasted image 20210609194117.png)

Before clicking generate we must listen on port 4444 port we set earlier.

![](Pasted image 20210610184955.png)

Got a hit as user kid in the box. Spawnning a tty shell using python3

`python3 -c 'import pty; pty.spawn("/bin/bash")'`

# Privilege Escalation

On further exploration we an intersting file name `scanlosers.sh` in user pwn's home directory

![](Pasted image 20210610185350.png)


```bash
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
```

On analyzing the file we could find it is taking `/home/kid/logs/hackers` as input and cutting in the contents in the file upto field 3 and tries to run nmap aganist the result . Lets try to do something to get some reverseshell as the user pwn with hackers file.

lets curl to port 9001 on our machine by putting code in hackers file.

![](Pasted image 20210610191728.png)

We got a hit with curl on port 9001 as expected.

![](Pasted image 20210610191607.png)

Let's upgrade by placing reverse shell

![](Pasted image 20210610192817.png)

got a hit as user pwn . user pwn has permission to run msfconsole as root without password

![](Pasted image 20210610193106.png)

with msfconsole we can acess root folder and read root flag

![](Pasted image 20210610193249.png)