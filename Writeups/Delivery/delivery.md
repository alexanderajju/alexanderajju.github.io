---
permalink: /delivery
layout: encrypted_post
passphrase: test
title: delivery
---

Monitors was a Hard difficulty Linux box created by TheCyberGeek. Initial foothold on the box was to find a vulnerable wordpress plugin with a LFI, using which we can read file descriptor and read apache logs. From the logs we get a new subdomain (Host) on the box for cactic, Also using the LFI we can read wp-configs.php which contained a credentials. Using Credentials Spraying we find that is the admin Credentials for cactic. Checking the Version for exploits we find that version of cactic is Vulnerable to SQLi which can be converted to RCE. Using which we get a shell on the box as www-data. Enumerating the box we find a service from crontab, following that we find credential for the user of the box. Enumerating the box as user marcus we see a notes.txt in his home directory. Following that hint we find a weird local only service running. port forwarding that service we discover that is an apache ofbiz. Trying a msf module for that give us a root shell in an docker container. Enumerating that we see we have CAP_SYS_MODULE capability and following Hacktrick docker escape we can get a root shell on the box.
