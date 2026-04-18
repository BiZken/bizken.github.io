
---
layout: ../../layouts/Post.astro
title: "Exfiltratören"
description: "Looking through a pcap file to identify the exfiltration happening on the network"
type: writeup
platform: FRA
tags: ["Network", "exfiltration", "Forensics", "wireshark"]
backLink: /writeups
backLabel: Back to writeups
---


# Exfiltratören — TryHackMe Writeup

**Platform:** FRA  
**Category:** SOC / network Forensics/ Wireshark  
**Tools Used:** Wireshark, Base45  

---

## Scenario

Yta 0x33 AB, a secretive company in the space industry, suspects that one of its employees is leaking sensitive information to outsiders. It is a serious situation, as there are indications that the employee is an agent for an actor planning to carry out a break-in at the company. Analyze the traffic and try to find what information has been exfiltrated.

---

## Flags
This is not the type of CTF you find at platform like THM or Hackthebox, you don't have anything to go on.

All you get is a pcap file with network traffic so the first thing I did was to check some basic stats, like all the IP addresses in the pcap.
<img src="/exfil/step1.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Flag 1:
After checking some of the statistics to create an image of whats going on I started to look at the trafic. I like to follow streams whenever I'm working with pcap and I checked the first TCP stream which was FTP communication. As soon as I opened it I got greeted by a skull. There are no clear flags here all we have is a password and a sign of a file called "msg.txt"
So I went to the next TCP stream where we can find a message that is base64 encoded. Decoding this we get the flag

***flagga{functional_treshold_power}***
<img src="/exfil/FTPscary.png" alt="Res1" style="max-width: 500px; width: 100%;">





### Flag 2:
***flagga{qrazy_basy}***
Going to the next TCP stream we have SMTP traffic where two files has been sent and they have been encoded to base64. Taking these base64 encoded files and making them into files again we get an image aswell as a pdf file. 
<img src="/exfil/PDF.png" alt="Res1" style="max-width: 500px; width: 100%;">

in the pdf file we have a qr code, after scanning the qr code we get a long string. At first I didn't know what the string was. I did try to use it as a password with steghide on the image we also got but no success. Instead after some testing I put it through a cipher detector where base45 gave a high probability score. Decoding it with Base45 we can see that the first line says "PNG" so I took the base45 string and decoded it into a PNG file that gave us the flag
<img src="/exfil/2.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/exfil/2.2.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/exfil/2_final.png" alt="Res1" style="max-width: 500px; width: 100%;">

<img src="/exfil/image.png" alt="Res1" style="max-width: 500px; width: 100%;">


### Flag 3:

In the PCAP file there are alot of different types of traffic, one is ICMP and i started going through it but couldn't find anything. From previous invetigations I found that 10.0.0.10 has sent alot of traffic so I filtered by it as source. Nothing came up clearly but after being bored and spamming the arrow keys i found that each packet sent have in the header a character. Going through them one by one you get a flag.

***Flagga{exf1l_by_p1ng}***

<img src="/exfil/ICMP.png" alt="Res1" style="max-width: 500px; width: 100%;">



### flag n:
there are more flags in the pcap but for now I have not found them. There are more traffic types That I will take a look at like fragmented UDP that can be used for exfiltration.

There are some hint That I will check, like the msg.txt file and the code on the flag 2 image


---
