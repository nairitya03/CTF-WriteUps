![1](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/1.png width='200' height='300' style='centerme')
# Cyborg
Link: <https://tryhackme.com/room/cyborgt8> 
By [@fieldraccoon](https://twitter.com/fieldraccoon)

A box involving encrypted archives, source code analysis and more.
----------------------------------------------------------
### Lets get Started...

####  1. Nmap
So first start with a simple nmap scan to know what ports are open and what
services are running on _ > target_Machine _.
```bash 
$ sudo nmap -T4 -sC -sS -A <machine_ip> >> nmap.out 
```

![2](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/2.png)

So ports 22 & 80 are open. Lets visit the webpage on port 80 and it was just a
default Apache2 webpage. Didn’t give out much.

![3](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/3.png)


#### 2. Gobuster
So lest brute force possible directories on webserver publicly accessible
```bash 
$ sudo gobuster dir -u 10.10.46.127 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -o dir.out
```

![4](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/4.png)

We got _/admin_ and _/etc_ directory lets explore them and > Boom!! we have
something interesting,A Password hash, Save it for later use

![5](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/5.png)

Further more poking found a public archive.tar file.

![6](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/6.png)


#### 3. Password Cracking
So we found a hash before lets crack it. John the Ripper is a great tool for it.
```bash 
$ john --fork=4 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

![7](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/7.png)

#### 4. Research
Now lets take a look at archive.tar file, extract it with tar 
```bash 
$ tar -xf archive 
```
Exploring the extracted files revealed a Readme file.

![8](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/8.png)

After a little research I found about borg, what it is and how it is used.
So as its borg archive unpack it using borg to a directory unziped.
```bash
$ borg mount home/field/dev/final_archive unziped
``` 
It seems it requires a password, lets try enter the one we found earlier, And dig for something useful.
There we have it a username and password. Lets ssh in target machine.

![9](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/9.png)
![10](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/10.png)


#### 5. Gaining Access
ssh into target with the username and password found and there we have it
our user flag.

![11](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/11.png)


#### 6. Privilege Escalation
Check if alex user is in sudoers list 
```bash
$ sudo -l
```
Seems like there is nopasswd sudo access on _backup.sh_ file. Lets exploit it.

![12](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/12.png)


#### 7. Source Code Analysis
Lets read whats happening in backup.sh

![13](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/13.png)

And script has a small chunk of code which seems to take input with a flag c
and echo it, basically it can run bash commands.

![14](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/14.png)

With that > Cyborg is rooted.
