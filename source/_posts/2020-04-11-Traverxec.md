---
title: Traverxec HTB
date: '2020-04-11 21:50:00'
categories: ["htb"]
---

## User

First I visit port 80/http I see a static website, nothing special. Before trying to run gobuster, I visit /robots.txt and I get a error, but I see that the server is not apache nor nginx.
The web server is *nostromo* version 1.9.6. I use `searchsploit nostromo` and find that there is a rce. Using metasploit I get a reverse shell. Then I run another command to get a proper reverse shell.
Inside /var/nostromo/conf I have the configuration files of the web server. The nhttpd.conf contains the path to the homedirs and other configurations regarding the  web server. I read the manual and get a overview of the config. The homedirs_public is `public_www` which means there is a folder *public_www* inside the users home directory. I visit the home directory and see that there is david folder. I cd into the folder, but I can't list any files. Now I cd again into public_www and there I can list files. I download the gzip archive inside protected-file-area. I extract the archive and  get a /home dir with .ssh folder inside containing the public and private ssh key. The private key is encrypted so I use ssh2john and find the password. Using the password I can ssh into the server, and grab user.txt

### Root

Nothing of much interest returned from linenum.sh. Manually I find a shell script located at /bin on my home dir.
```
#!/bin/bash

cat /home/david/bin/server-stats.head
echo "Load: `/usr/bin/uptime`"
echo " "
echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -l`"
echo " "
echo "Last 5 journal log lines:"
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat 
```
At the last line of this script I see a command running as root, showing last 5 lines of log, as comment also says. And then is piped to cat. I go to *GTFOBins* and see If I can do something with journalctl but, the script executs and then ends, so I can't enter `!/bin/sh` and get shell as root. From GTFOBins I read **This invokes the default pager, which is likely to be less, other functions may apply.** 

## What is a pager?

`A pager is a piece of software that helps the user get the output one page at a time,by getting the size of rows of the terminal and displaying that many lines.`

But In my case the less which is the default pager isn't executed. I read the man page of journalctl and find out that `If no pager implementation is discovered no pager is invoked. Setting this environment variable to an empty string or the value "cat" is equivalent to passing --no-pager`. So the reason for not getting less work is because it is disabled because the command is piped to cat. I now copy the script to another file and remove `| /usr/bin/cat`, so the pager is set back to less. But when I run it I get no `less` working. Again from man pages I found out that `The output is paged through less by default, and long lines are "truncated" to screen width. The hidden part can be viewed by using the left-arrow and right-arrow keys. Paging can be disabled; see the --no-pager option and the "Environment" section below.` So I need to resize my terminal so less get executed. After that I run the script again and get less executed, and then use `!/bin/sh` to get a shell as root.
