# SCANNING
-> nmap -sC -sV 10.10.242.203
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:07:96:0d:c4:b6:0c:d6:22:1a:e4:6c:8e:ac:6f:7d (RSA)
|   256 ba:ff:92:3e:0f:03:7e:da:30:ca:e3:52:8d:47:d9:6c (ECDSA)
|_  256 5d:e4:14:39:ca:06:17:47:93:53:86:de:2b:77:09:7d (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

# ENUM

## enum dir
-> gobuster dir -e -u http://10.10.242.203/ -w /usr/share/wordlists/dirb/common.txt
http://10.10.242.203/index.html           (Status: 200) [Size: 57]

-> gobuster dir -e -u http://cyprusbank.thm/ -w /usr/share/wordlists/dirb/common.txt
http://cyprusbank.thm/index.html           (Status: 200) [Size: 252]

## enum dns
-> dnsmap cyprusbank.thm -w /usr/share/dnsmap/wordlist_TLAs.txt  
 admin.cyprusbank.thm
 
# VISITING THE ADMIN SITE
login with the given creds: Olivia Cortez:olivi8

# SCAN ON ADMIN SITE
-> nmap -sC -sV admin.cyprusbank.thm  
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:07:96:0d:c4:b6:0c:d6:22:1a:e4:6c:8e:ac:6f:7d (RSA)
|   256 ba:ff:92:3e:0f:03:7e:da:30:ca:e3:52:8d:47:d9:6c (ECDSA)
|_  256 5d:e4:14:39:ca:06:17:47:93:53:86:de:2b:77:09:7d (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
| http-title: Cyprus National Bank
|_Requested resource was /login

-> gobuster dir -e -u http://admin.cyprusbank.thm/ -w /usr/share/wordlists/dirb/common.txt
http://admin.cyprusbank.thm/login                (Status: 200) [Size: 2195]
http://admin.cyprusbank.thm/Login                (Status: 200) [Size: 2195]
http://admin.cyprusbank.thm/logout               (Status: 302) [Size: 28] [--> /login]
http://admin.cyprusbank.thm/messages             (Status: 302) [Size: 28] [--> /login]
http://admin.cyprusbank.thm/search               (Status: 302) [Size: 28] [--> /login]
http://admin.cyprusbank.thm/Search               (Status: 302) [Size: 28] [--> /login]
http://admin.cyprusbank.thm/settings             (Status: 302) [Size: 28] [--> /login]

## look for js/html files
-> gobuster dir -e -u http://admin.cyprusbank.thm/ -w /usr/share/wordlists/dirb/common.txt -x js
none

try on login
-> gobuster dir -e -u http://admin.cyprusbank.thm/login/ -w /usr/share/wordlists/dirb/common.txt -x js
none

look for html
-> gobuster dir -e -u http://admin.cyprusbank.thm/ -w /usr/share/wordlists/dirb/common.txt -x html 
none

# VIISTING SITE
"http://admin.cyprusbank.thm/messages/?c=5" has a counter, lets mess with that

**found**:
DEV TEAM: Thanks Gayle, can you share your credentials? We need privileged admin account for testing
Gayle Bev: Of course! My password is 'p~]P@5!6;rs558:q'

**new creds**:
Gayle Bev
p~]P@5!6;rs558:q

**he has admin priviliges**, so we can access customer settings and look for user details

- changed the password for Tyrell Wellick to: 'newpassword' (!!! did not work !!!)
- looked for Tyrell's phone number

Tyrell's phone number: **842-029-5701**

# PHASE 2: ACCESS THE MACHINE
# ...
