# NMAP
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
5083/tcp open  unknown

# TRY TO ACCESS MYQSL
-> mysql --host=10.10.253.83     
ERROR 2002 (HY000): Received error packet before completion of TLS handshake. The authenticity of the following error cannot be verified: 1130 - Host 'ip-10-21-217-122.eu-west-1.compute.internal' is not allowed to connect to this MariaDB server

# DIRB ON WEB SERVER
+ http://10.10.253.83:80/akeeba.backend.log (CODE:403|SIZE:277)             
+ http://10.10.253.83:80/development.log (CODE:403|SIZE:277)                
+ http://10.10.253.83:80/index.php (CODE:302|SIZE:1)                        
+ http://10.10.253.83:80/production.log (CODE:403|SIZE:277)                 
+ http://10.10.253.83:80/robots.txt (CODE:200|SIZE:37)                      
+ http://10.10.253.83:80/server-status (CODE:403|SIZE:277)                  
+ http://10.10.253.83:80/spamlog.log (CODE:403|SIZE:277) 

# DEEPER NMAP
-> nmap -A -p22,80,3306,5038 10.10.253.83
22/tcp   open  ssh      OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
| ssh-hostkey: 
|   256 bf:39:86:4f:93:47:4f:16:8b:52:ab:0b:cd:4d:c6:92 (ECDSA)
|_  256 ca:ab:e6:3b:3d:b0:ad:88:4e:42:21:c7:73:73:77:08 (ED25519)
80/tcp   open  http     Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
| http-title:             MagnusBilling        
|_Requested resource was http://10.10.121.159/mbilling/
3306/tcp open  mysql    MariaDB 10.3.23 or earlier (unauthorized)
5038/tcp open  asterisk Asterisk Call Manager 2.10.6

# METASPLOIT ASTERISK
auxiliary/gather/asterisk_creds
nothing

# METASPLOIT MAGNUSBILLING
exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258
