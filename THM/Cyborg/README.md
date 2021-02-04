<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/1.png" width="350" ></p>

# Cyborg
Link: <https://tryhackme.com/room/cyborgt8> 
By [@fieldraccoon](https://twitter.com/fieldraccoon)

A box involving encrypted archives, source code analysis and more.
----------------------------------------------------------
### Lets get Started...

####  1. Nmap
So first start with a simple nmap scan to know what ports are open and what
services are running on Target_Machine.
```bash 
$ sudo nmap -T4 -sC -sS -A <machine_ip> >> nmap.out 
```

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/2.png" width="550" ></p>

So ports 22 & 80 are open. Lets visit the webpage on port 80 and it was just a
default Apache2 webpage. Didn’t give out much.

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/3.png" width="550" ></p>


#### 2. Gobuster
So lest brute force possible directories on webserver publicly accessible
```bash 
$ sudo gobuster dir -u 10.10.46.127 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -o dir.out
```

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/4.png" width="550" ></p>

We got _/admin_ and _/etc_ directory lets explore them and > Boom!! we have
something interesting,A Password hash, Save it for later use

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/5.png" width="550" ></p>

Further more poking found a public archive.tar file.

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/6.png" width="550" ></p>

#### 3. Password Cracking
So we found a hash before lets crack it. John the Ripper is a great tool for it.
```bash 
$ john --fork=4 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/7.png" width="550" ></p>

#### 4. Research
Now lets take a look at archive.tar file, extract it with tar 
```bash 
$ tar -xf archive 
```
Exploring the extracted files revealed a Readme file.

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/8.png" width="550" ></p>

After a little research I found about borg, what it is and how it is used.
So as its borg archive unpack it using borg to a directory unziped.
```bash
$ borg mount home/field/dev/final_archive unziped
``` 
It seems it requires a password, lets try enter the one we found earlier, And dig for something useful.
There we have it a username and password. Lets ssh in target machine.

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/9.png" width="400" ></p>
<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/10.png" width="400" ></p>


#### 5. Gaining Access
ssh into target with the username and password found and there we have it
our user flag.

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/11.png" width="550" ></p>


#### 6. Privilege Escalation
Check if alex user is in sudoers list 
```bash
$ sudo -l
```
Seems like there is nopasswd sudo access on _backup.sh_ file. Lets exploit it.

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/12.png" width="550" ></p>


#### 7. Source Code Analysis
Lets read whats happening in backup.sh

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/13.png" width="400" ></p>

And script has a small chunk of code which seems to take input with a flag c
and echo it, basically it can run bash commands.

<p align="center"> <img src="https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/Cyborg/Screenshots/14.png" width="550" ></p>

With that Cyborg is rooted.
