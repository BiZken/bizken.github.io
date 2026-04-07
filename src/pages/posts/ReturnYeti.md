---
layout: ../../layouts/Post.astro
title: "Return Of Yeti"
description: "The goal is to analyze wireless pcapng. Looking for signs of intrusion as well as doing some offensive techniques."
type: writeup
platform: THM
tags: ["Linux Logs", "Forensics", "Linux"]
backLink: /writeups
backLabel: Back to writeups
---


# New Hire Old Artifacts — TryHackMe Writeup

**Platform:** TryHackMe  
**Category:** Forensics and offensive 
**Difficulty:** Hard  
**Tools Used:** Linux CLI, aircrack, airdecap , wireshark, pyrdp

---

## Scenario

The goal is to analyze wireless pcapng. Looking for signs of intrusion as well as doing some offensive techniques. 

---

## Flags

### Flag 1:
_What's the name of the WiFi network in the PCAP?_ 

***ANSWER:*** `FreeWifiBFC`

Opening the _pcapng_ file the first answer can be seen immediately by looking for the SSID.


<img src="/returnyeti/1.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Flag 2:
_What's the password to access the WiFi network?_

***ANSWER:*** `Christmas`


The _pcapng_ is of wireless communication that is protected by encryption however this is crackable as long as you have captured the 4 way EAPOL handshake
searching for `eapol` in wireshark and we can see that the EAPOL handshake is in the pcapng file

now with this we can use `aircrack-ng` to get the password, I used rockyou wordlist for this as its always the start for me.

I started with trying to crack the `pcapng` file however it failed as it didn't find any networks in the file
So I saved it as a `pcap` file and tried again with success for the password _Christmas_.
```Bash
aircrack-ng advctf.pcapng -w /usr/share/Seclists/rockyou.txt
```
<img src="/returnyeti/2.png" alt="Res1" style="max-width: 500px; width: 100%;">



### Flag 3:
_What suspicious tool is used by the attacker to extract a juicy file from the server?_

***ANSWER:*** ``

So now when we have the password we need to decrypt it, using `airdecap-ng` creates a new `pcap` file that has been decrypted
Providing the SSID from the first question and the key from the second

```Bash
airdecap-ng advctf.pcap -e FreeWifiBFC -p Christmas
```
_-e_ is for the SSID
_-p_ Password for the SSID
<img src="/returnyeti/3.png" alt="Res1" style="max-width: 500px; width: 100%;">

opening the decrypted file we can actually see whats going on. There are a lot of packets in the file and after a quick look one thing to notice is the possible portscan.
Not the question but a good note incase something comes later.
<img src="/returnyeti/3-portscan.png" alt="Res1" style="max-width: 500px; width: 100%;">

After following some tcp streams I saw a lot of communication has been sent to port 4444 and following that TCP stream I found a clear sign of a shell with some basic recon on a system being done.
<img src="/returnyeti/4444.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/returnyeti/3-signofshell.png" alt="Res1" style="max-width: 500px; width: 100%;">

looking at the commands send we can see mimikatz being used. They encoded the file with base64
<img src="/returnyeti/mimi.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/returnyeti/base64.png" alt="Res1" style="max-width: 500px; width: 100%;">




### Flag 4
_What is the case number assigned by the CyberPolice to the issues reported by McSkidy?_

***Answer:*** `31337-0`

This is the part it took a lot longer to figure out what to do, I checked some of the streams in wireshark but nothing. And I remembered the previous question and decided to recreate that file.
I was not able to extract it from within wireshark but they encoded the file with base64 with powershell so what I did was to just send the encoded base64 string back into the same file name and extension as before.
I saw that it was a `.pfx` file I had no clue what it was but a quick google search and apparently its a `password-protected file used to securely store digital certificates, their corresponding private keys, and intermediate certificates`
<img src="/returnyeti/bashfor4.png" alt="Res1" style="max-width: 500px; width: 100%;">

After I recreated it I didn't know what to do, no clue so I went  back to wireshark trying to find anything and after more time than I'd like to admit, I saw RDP traffic and looked up if there is anything with RDP and PFX and
apparently you can use PFX to have the keys to encrypt the RDP sessions. Also in the wireshark TCP stream I saw that the file came from `Remote Desktop`.

I found that I can "see" the file by using `openssl`:
```Bash
openssl pkcs12 -info -in file.pfx
```
Running the info for the file you are prompted for a password, thought it was the same as for the SSID but nope.

<img src="/returnyeti/4passwordask.png" alt="Res1" style="max-width: 500px; width: 100%;">

After some research I found that the default password is _mimikatz_ when you use mimikatz and after that it worked.

<img src="/returnyeti/4afterpassword.png" alt="Res1" style="max-width: 500px; width: 100%;">

getting the key from the file I used:
`openssl pkcs12 -in file.pfx -nocerts -out key.key -nodes`
Again the password is _mimikatz_
And I thought all I needed to do was to import this into wireshark but I messed up, 

<img src="/returnyeti/felkey4.png" alt="Res1" style="max-width: 500px; width: 100%;">

its "pem" and not "key" also there is an additional step needed
from the `.pem` File I need to create an RSA key, so I renamed it to _key.pam_ and ran:
`openssl rsa -in key.pam -o key.key`

This key was then imported into wireshark under TLS settings with the source as _10.1.1.1_.


Next we needed to figure out how to get the case number. Since everything so far has been about RDP I figured it got to have something to do with that.
Looking through the RDP entries in wireshark got me nowhere but I found a way to replay RDP sessions with a tool called `pyrdp`. After fighting with some version compatibility I decided to just run it in docker.

First I extracted all RDP from wireshark and put it through the `pyrdp` but it didn't work, it had no info so then I just used the full layer 7 extraction from wireshark and tested again which showed the RDP session. the screen as well as what was typed. Its also shows the next questions answer as well. 
<img src="/returnyeti/finals.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/returnyeti/finals2.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Flag 4
_What is the content of the yetikey1.txt file?_

***Answer:*** `1f9548f131522e85ea30e801dfd9b1a4e526003f9e83301faad85e6154ef2834`


---
