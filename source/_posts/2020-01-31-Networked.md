---
title: Networked HTB
date: '2020-01-31 20:59:00'
categories: ["htb"]
---

On port 80 there is a webserver, running gobuster we find /backup. Download the backup.tar. See the source code, but also the filenames of some of the web apps functionallity.
On upload.php, we can upload an image($validext = array('.jpg', '.png', '.gif', '.jpeg');) which we can view it later on photos.php.
We can intercept the connection with burp and upload a valid image, before letting the request go through the webserver we add a simple php rce script code at the end of the body, also we change the filename to imagename.php      .jpg
Visit the upload image at /upload/imagename, you see the code but not the image, now use ?cmd=*$urlencodedrevershellcommand* to get a shell.

User:
Inside the guly home folder there is a script called check_attack.php which runs every 3 minutes(based on crontab.guly file inside the same dir).
This script checks for bad filenames uploaded at /uploads/. We can create a file with ;$revshell, and get reverse shell as guly.
I can read user.txt now.

Root:
Running linenum.sh it finds a shell script located at /usr/local/sbin/changename.sh, which script asks us about network config information and writes them to a file. We cannot use ; or a special character like $() && &, in order to inject code.
I create a script with code for reverse shell and save it to /tmp/, On every question asked by the script I give it bash /tmp/revshell Ps: . not allowed. And get reverse shell as root.

Lessons Learned:
~Regex is always usefull.
~Be aware of what file types you allow.
