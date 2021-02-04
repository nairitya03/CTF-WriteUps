# > 		Chocolate Factory
<https://tryhackme.com/room/chocolatefactory>
----------------------------------------------
![1](url)

So lets Dive in…
First copy the _Machine IP_ in a text file target.txt

Began with a nmap scan..```bash #sudo nmap -T4 -sV -sC target.txt ```
And with that we get some interesting results.

![2](url)

Just click the link and it seems its downloadable.

Now open and it seems to be a different encoding probably Hex. Lets try
reading the file as strings ```bash #strings downloaded_file.txt ```

![3](url)

There’s also a FTP port open lets play with it…
```bash #ftp target.txt ``` and get the file on it.

![4](url)

It seems it is Stego image. Lets check if it has file encoded in it…
```bash #steghide -info gum_room.jpg ``` and yep it had some file in it.
So lets extract it. ```bash #steghide --extract -sf gum_room.jpg ```

![5](url)

Extracted file seems to be some kind of encryption so lets jump to
[Cyberchef]( https://gchq.github.io/CyberChef/ ), upload &
decrypt it and save output to a file hash.txt

![6](url)

It looks like user and password hashes, So lets crack hash…
```bash #sudo john --wordlist=/usr/share/wordlist/rockyou.txt hash.txt ```

![7](url)

Now that’s done lets login to webpage using cracked password…
This give us with a dashboard that run commands.

![8](url)

Now lets get a reverse shell on the target. And then stabilize shell.

![9](url)

### Now lets try to get charlie user...

After lurking around I found some interesting files.SSH to charlie with the private RSA key and get user flag.

![10](url)


### Now lets get root user...
After some linux enumeration using [linEnum](https://github.com/rebootuser/LinEnum) or [linPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)
it was found there is a sudo pwnage with vim. 

![11](url)

Root flag is a python file and seems require a key as input.
After some hit and trial I decided to use the key found before from web as
input and Woaah!! There it was.

![12](url)

PS: I was unable to make that python script to run successful, I got an
alternative way to decode it (Online Fernet Decoder).

And with that _Chocolate Factory_ is Rooted...