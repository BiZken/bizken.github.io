
---
layout: ../../layouts/Post.astro
title: "IronShade"
description: "Perform a compromise assessment on a Linux host and identify the attack footprints, Investigate the server and identify the footprints left behind after the exploitation."
type: writeup
platform: THM
tags: ["Linux Logs", "Forensics", "Linux"]
backLink: /writeups
backLabel: Back to writeups
---


# New Hire Old Artifacts — TryHackMe Writeup

**Platform:** TryHackMe  
**Category:** SOC 
**Difficulty:** Medium  
**Tools Used:** Linux CLI  

---

## Scenario

The goal is to identify artifacts left behind by a threat actor, looking for backdoor accounts, persistence mechanisms, suspicious processes, and malicious packages by performing live forensic analysis directly on the system.

---

## Flags

### Flag 1:
_What is the Machine ID of the machine we are investigating?_ 

***ANSWER:*** `dc7c8ac5c09a4bbfaf3d09d399f10d96`

checking the machine ID on a linux machine you can just read the /etc/machine-id

```Bash
cat /etc/machine-id
```

<img src="/ironshade/1.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Flag 2:
_What backdoor user account was created on the server?_

***ANSWER:*** `mircoservice`

<img src="/ironshade/2.png" alt="Res1" style="max-width: 500px; width: 100%;">

All local user accounts on a Linux system are stored in `/etc/passwd`. By looking the bottom of this file, you can find user-created accounts and not system accounts user accounts typically have a `/home/` directory. 

```Bash
cat /etc/passwd
```

### Flag 3:
_What is the cronjob that was set up by the attacker for persistence?_

***ANSWER:*** `@reboot /home/mircoservice/printer_app`

Cronjobs are scedualed tasks that can be started by users and set to run every X. Listing all the cronjobs you need to see the `crontab -l` command. Checking for the user from question 2 you can se the `printer_app` set to run when system boots meaning persistance.

```Bash
sudo crontab -l
```

<img src="/ironshade/3.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Flag 4
_Examine the running processes on the machine. Can you identify the suspicious-looking hidden process from the backdoor account?_

***Answer:*** `.strokes`

Looking for the processes I first ran _htop_ as it is something regularly do on my own computer but for checking resource usage of the thing running. This showed `/home/mircoservice/.tmp/.strokes` 

<img src="/ironshade/4_error.png" alt="Res1" style="max-width: 500px; width: 100%;">

but I wanted to be more specific so I ran _lsof_ on the users home directory, its lists open files and then I saw more clearly the _.strokes_. There were two files open/running but since it was supposed to be a hidden file it need the have the dot (.) infront of it. 

```Bash
sudo lsof +D /home/mircoservice
```

<img src="/ironshade/4.png" alt="Res1" style="max-width: 500px; width: 100%;">



### Flag 5
_How many processes are found to be running from the backdoor account’s directory?_

***Answer:*** `2`

The `lsof` output from the previous question revealed two  processes running from the backdoor account's directory. Using `lsof +D` recursively scans subdirectories, which is why it caught both.

### Flag 6
_What is the name of the hidden file in memory from the root directory?_

***Answer:*** `.systmd`

Looking at the _/root_ directory for hidden files I ran

```Bash
sudo ls -la /root
```
 This showed nothing unusual — just standard dotfiles like `.bashrc` and `.profile`. The key was realizing the question asks about the *root directory* (`/`), not the root user's home (been doing too much red team). Checking the filesystem root revealed a hidden directory `.systmd`

<img src="/ironshade/6.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Flag 7
_What suspicious services were installed on the server? Format is service a, service b in alphabetical order._

***Answer:*** `backup.service, strokes.service`

To see the installed services I ran _systemctl_ with the flag to just show services:

```Bash
systemctl --type=service
```

its quite the list but one was easy to find as it hade the "strokes" name from previous questions, however the second one was a bit harder, I ended up not looking at the name but the other info to see if I could find something odd and I saw `SUB` being _start-pre start_ aswell as a one word description for a `UNIT` called _backup.service_ Turns out after some research that this is not a standard service so this was the second one.

<img src="/ironshade/7.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Flag 8
_Examine the logs; when was the backdoor account created on this infected system?_

***Answer:*** `Aug  5 22:05:33`

The `auth.log` records all authentication-related events, including account creation via `useradd` or `adduser`. Grepping for the backdoor username and looking at the earliest entry gives the creation timestamp..

```Bash
cat /var/log/auth.log | grep -a mircoservice
```
_The `-a` flag tells `grep` to treat the file as text even if it contains binary data which the lohfile did_

<img src="/ironshade/8.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Flag 9
_From which IP address were multiple SSH connections observed against the suspicious backdoor account?_

***Answer: *** `10.11.75.247` 

Continuing through the `auth.log` output from the previous grep, the SSH connection entries clearly show `10.11.75.247` as the source IP for logins to the `mircoservice` account

<img src="/ironshade/10.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Flag 10
_How many failed SSH login attempts were observed on the backdoor account?_

***Answer:*** `8`

Adding a another grep the question 9s log search for "failed" shows all the failed password attempt to the user. It showed 6 lines however two of the lines were "message repated 2 times" so they count as 2/line

```Bash
cat /var/log/auth.log | grep -a mircoservice | grep -i "failed"
```
_the -i flag is so grep is using incasesentivive_

<img src="/ironshade/11.png" alt="Res1" style="max-width: 500px; width: 100%;">


### Flag 11
_Which malicious package was installed on the host?_

***Answer:*** `pscanner`

All the packages installed can be seen using the 
`apt-mark showmanual_`
With the resulting packages all I did was to exclude the packages that I know comes with a linux distro. Resulting in that I found _pscanner_, sounding suspicious and with a quick search I found that it is used for scanning TCP ports.


<img src="/ironshade/13.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Flag 12
_What is the secret code found in the metadata of the suspicious package?_

***Answer:*** `{_tRy_Hack_ME_}`

Looking at the metadata for the package I used:
```Bash
dpkg -l | grep pscanner
```
Too get extra information about the package and there the "secret_code" was.


<img src="/ironshade/last.png" alt="Res1" style="max-width: 500px; width: 100%;">


---
