![1](url)
# > Cyborg
Link:<https://tryhackme.com/room/cyborgt8> By [@fieldraccoon](https://twitter.com/fieldraccoon)

A box involving encrypted archives, source code analysis
and more.
----------------------------------------------------------
### Lets get Started...

####  1. Nmap
So first start with a simple nmap scan to know what ports are open and what
services are running on _ > target_Machine _.
```bash $ sudo nmap -T4 -sC -sS -A <machine_ip> >> nmap.out ```

![2](url)

So ports 22 & 80 are open. Lets visit the webpage on port 80 and it was just a
default Apache2 webpage. Didn’t give out much.

![3](url)


#### 2. Gobuster
So lest brute force possible directories on webserver publicly accessible
```bash $ sudo gobuster dir -u 10.10.46.127 -w
/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -o dir.out ```

![4](url)

We got _/admin_ and _/etc_ directory lets explore them and > Boom!! we have
something interesting,A Password hash, Save it for later use

![5](url)

Further more poking found a public archive.tar file.

![6](url)


#### 3. Password Cracking
So we found a hash before lets crack it. John the Ripper is a great tool for it.
```bash $ john --fork=4 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt ```

![7](url)

#### 4. Research
Now lets take a look at archive.tar file, extract it with tar ```bash $ tar -xf archive ```
Exploring the extracted files revealed a Readme file.

![8](url)

After a little research I found about borg, what it is and how it is used.
So as its borg archive unpack it using borg to a directory unziped.
```bash $ borg mount home/field/dev/final_archive unziped ``` and seems it require a
password, lets enter the one we found earlier. And dig for something useful.
And there we have it a username and password. Lets ssh in target machine.

![9](url)
![10](url)


#### 5. Gaining Access
ssh into target with the username and password found and there we have it
our user flag.

![11](url)


#### 6. Privilege Escalation
Check if alex user is in sudoers list ```bash $ sudo -l ```.
Seems like there is nopasswd sudo access on _backup.sh_ file. Lets exploit it.

![12](url)


#### 7. Source Code Analysis
Lets read whats happening in backup.sh

![13](url)

And script has a small chunk of code which seems to take input with a flag c
and echo it with $ , basically it can run bash commands.

![14](url)

With that > Cyborg is rooted.