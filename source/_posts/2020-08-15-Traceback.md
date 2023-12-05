---
title: Traceback HTB Writeup
date: '2020-08-15 19:50:00'
author: fuxsocy
description: Traceback By Fuxsocy
categories: ["htb"]
banner_img: /img/Traceback.png
---

### User

On port 80 we see a pwned website. On the comment of the html source code we see something about webshell also the webpage itself tells us about a backdoor. After visiting the box creator github repo, I find a repo with web shells.I get these web shells names and then use them with wfuzz.

```
git clone https://github.com/Xh4H/Web-Shells

```

`ls -la Web-Shells | awk '{print $9}' > potentialshells.txt`

Then give the filenames to wfuzz. Shell with name _smevk.php_ found. admin, admin credentials and then `execute` command for a reverse shell.

After running `sudo -l` I can as user sysadmin /home/webadmin/luvit. Running luvit like `sudo -u sysadmin /home/webadmin/luvit` gives me an interactive lua prompt. I can execute command with `os.execute('bash /tmp/add.sh')`  and then ssh on user sysadmin. User gained.


### Root

I upload pspy64s on the machine and see that the root user every minute copies from `/var/backups/.update-motd.d/*` to `/etc/update-motd.d/`. Then executes `/bin/sh /etc/update-motd.d/91-release-upgrade on that folder`. Since I have rights to write the files inside `/etc/update-motd.d/`. I try to add on every file and see if I can get to add my ssh public key to root user since root is allowed to ssh into the box. I give this command. `while true; do echo "echo 'mysshkey' >> /root/.ssh/authorized_keys" >> /etc/update-motd.d/10-help-text` and then watch the file live as it get's writen by backup but before getting execute I add my command and the it executes as root, adding my ssh key. I then ssh and get root.

