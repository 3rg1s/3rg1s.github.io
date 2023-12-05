---
title: OpenAdmin HTB Writeup
date: '2020-05-02 19:19:00'
categories: ["htb"]
banner_img: /img/OpenAdmin.png
---


## User.txt
From port scanning two ports are open `80` and `22`. I brute force the root directory  / using gobuster and find out /ona/ visiting this /ona/ I see it is openadmin. I search using `searchsploit` and found out a shell script which gives me rce. Using `dos2unix` I convert this script to unix format to prevent errors. Using the script I can run commands. I use wget and download a php reverse shell to the box. Then I run the script using php, and get a interactive shell. I look for configuration file to find possible passwords. After a little I find one at /var/www/ona/local and then use that password to su as jimmy(*found on /etc/passwd*). As jimmy user I can list all files inside internal folder because it's owned by jimmy. I find some php files there. `index.php`, `main.php` and `logout.php`. Inside `index.php` a function checks if the user is `jimmy` and the password is same as a sha512sum of the password we give and if succeded we get ssh private key for joanna user. I see if I can find the password on hashes.org and I found it to be `Revealed`. Now all I need is to access the website from my machine and then submit it. I run `netstat -alnp` and see that the site is on port `52846`. I create a ssh key pair on my machine and then  transfer the public key to .ssh/authorized_keys and ssh into the box. then using ENTER and ~C and then enter `-L 9001:localhost:52846`. Now on my host I visit http://localhost:52846 and using the form I submit the credentials and after I receive the ssh private key. I use ssh2john and rockyou.txt as wordlist, I crack the private key using john. Using the private key and the password I can now ssh as joanna, and get user.txt.

---------------------

## root.txt

I type `sudo -l` and find out that I can run `/bin/nano /opt/priv` on gtfo bins I found how I can get a shell from when I am inside nano. I run `sudo -u root /bin/nano /opt/priv` and then use 
```
^R^X
reset; sh 1>&0 2>&0
```
Now I am root.!
