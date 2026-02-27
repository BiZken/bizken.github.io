---
layout: ../../layouts/Post.astro
title: "PS Eclipse"
description: "Use Splunk to investigate the ransomware activity."
date: "2026-02-25"
type: writeup
platform: THM
tags: ["Forensics", "SOC", "Windows", "Splunk", "Blue Team", "SIEM"]
backLink: /writeups
backLabel: Back to writeups
---

## Overview

*_Splunk_* one of the most common SIEM solutions out there and its for a reason Splunk is powerful — but the learning curve is real especially the search syntax, you can make it super detailed and trim thousends of logs down to just what you want or keep it simple and be left with a few and manually go through them.
Let me be honest.

I often fight with the search syntax.

Splunk has that “it makes sense once you know it” energy. Until then, it feels like:

- Why didn’t that filter work?

- Why is this returning 30,000 events?

- Why does adding one space break everything?


This room revolves around:

- Windows event logs

- Sysmon telemetry

- PowerShell activity

- Suspicious executables

- Scheduled tasks and persistence

- Network connections to attacker infrastructure


## Flags

First before starting digging through logs the time frame was set to 16th may 2022 when the incident occured. Looking through all the indexes there are a total of _17078_ events.

<img src="/Pseclipse/Splunk.png" alt="description" style="max-width: 500px; width: 100%;">


### flag 1
_A suspicious binary was downloaded to the endpoint. What was the name of the binary?_

First task was pretty easy I just searched for any file with the extension ".exe" and too make sure that it is only files created I specified "EventCode=11" which is the sysmon ID for "File Created". Checking the top file names I found *_OUTSTANDING_GUTTER.exe_* in the \temp directory.


 
```
index=* EventCode=11 TargetFilename"*.exe"
```

***ANSWER:*** OUTSTANDING_GUTTER.exe

<img src="/Pseclipse/1.png" alt="description" style="max-width: 500px; width: 100%;">

### flag 2
_What is the address the binary was downloaded from? Add http:// to your answer & defang the URL._

Knowing the name of the file its easy to see every event related to the malicious files, just adding the name in the searchbar.

I started looking for different webpages that could have the file in the url for a download but that turned up fuck all. Realised then the name of the room and that it is about powershell so started to look out for powershell commands and found an base64 encoded powershell command tied to both the file aswell as the user. Also checked the first events as the download would occur early on.

Getting the URL I just decoded it in the [CyberChef](https://gchq.github.io/CyberChef/) with "From Base64" but also used #UTF-16LN# just to make it readable.

After that I Defanged the URL for the Task.
```
index=* OUTSTANDING_GUTTER.exe "DESKTOP-TBV8NEF\\keefan" | sort -_time desc
```

<img src="/Pseclipse/2_2_2.png" alt="description" style="max-width: 500px; width: 100%;">


<img src="/Pseclipse/2.png" alt="description" style="max-width: 500px; width: 100%;">


<img src="/Pseclipse/2_2.png" alt="description" style="max-width: 500px; width: 100%;">


***ANSWER:*** hxxp[://]886e-181-215-214-32[.]ngrok[.]io

### flag 3

_What Windows executable was used to download the suspicious binary? Enter full path._

Going back to the Splunk event you can see the "Image" which means the application used for said event. Obvious its powershell

***ANSWER:*** C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe

<img src="/Pseclipse/3.png" alt="description" style="max-width: 500px; width: 100%;">

### flag 4
_What command was executed to configure the suspicious binary to run with elevated privileges?_

For this you need to go back to the decoded powershell, I spent some time before I remembered that the command contained more than just the URL. Looking over the command again its using schedualed Tasks (schtasks) so for the search added the powershell path, user aswell as SCHTASKS:

```
index="main" "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe" User="DESKTOP-TBV8NEF\\keegan" SCHTASKS
```

Looking at the *_CommandLine_* there are 2 values, the first one shows the command for the configuration

***ANSWER:*** "C:\Windows\system32\schtasks.exe" /Create /TN OUTSTANDING_GUTTER.exe /TR C:\Windows\Temp\COUTSTANDING_GUTTER.exe /SC ONEVENT /EC Application /MO *[System/EventID=777] /RU SYSTEM /f

<img src="/Pseclipse/4.png" alt="description" style="max-width: 500px; width: 100%;">

<img src="/Pseclipse/4_answer.png" alt="description" style="max-width: 500px; width: 100%;">

### flag 5 
_What permissions will the suspicious binary run as? What was the command to run the binary with elevated privileges? (Format: User + ; + CommandLine)_

Looking at the command from the previous flag we se the *_/RU SYSTEM_* meaning the task will run as *_NT AUTHORITY\SYSTEM_*, this gives the executable full system access, therefor "elevated privileges".

***ANSWER:*** NT AUTHORITY\SYSTEM;"C:\Windows\system32\schtasks.exe" /Run /TN OUTSTANDING_GUTTER.exe

<img src="/Pseclipse/5.png" alt="description" style="max-width: 500px; width: 100%;">
<img src="/Pseclipse/5_2.png" alt="description" style="max-width: 500px; width: 100%;">

### flag 6
_The suspicious binary connected to a remote server. What address did it connect to? Add http:// to your answer & defang the URL._

Again having the filename in search bar but checking all the DNS Queries to get the domains with showed 5 events, knowing ngrok is a tunneling service and it was the latest event I figured it was the correct URL.

```
index="main" OUTSTANDING_GUTTER.exe QueryName="*"
```

***ANSWER:*** hxxp[://]9030-181-215-214-32[.]ngrok[.]io

<img src="/Pseclipse/6.png" alt="description" style="max-width: 500px; width: 100%;">

### flag 7
_A PowerShell script was downloaded to the same location as the suspicious binary. What was the name of the file?_

A powershell script was downloaded to the same directory as the malicious exe file meaning its in the \Temp directory so all needed to do was to search the \Temp directory for any powershell file (ps1 extension).

There were som policy scripts from what I assume microsoft but they usually don't send out scripts called _script.ps1_


***ANSWER:*** script.ps1

<img src="/Pseclipse/8.png" alt="description" style="max-width: 500px; width: 100%;">

### flag 8
_The malicious script was flagged as malicious. What do you think was the actual name of the malicious script?_

All files have a checksum usually used for integrity checks but you can also identify malicious files from known checksums of malicious files. Being on the same search as the previous question you can get the checksum of the _script.ps1_ file. Looking at virustotal you can search checksums to see if they are previously known for being malicious. Results from virus total [Here](https://www.virustotal.com/gui/file/e5429f2e44990b3d4e249c566fbf19741e671c0e40b809f87248d9ec9114bef9)

The Virustotal database identified the powershell script to be the _Blacksun_ ransomware

***ANSWER:*** BlackSun.ps1

<img src="/Pseclipse/8_2.png" alt="description" style="max-width: 500px; width: 100%;">

<img src="/Pseclipse/8_answer.png" alt="description" style="max-width: 500px; width: 100%;">

### flag 9
_A ransomware note was saved to disk, which can serve as an IOC. What is the full path to which the ransom note was saved?_

Going with the same approach as previous when it came to created file I used the _EventCode 11_. Since it is a note i assumed it is a txt file so I searched all files but the events had to have "txt" in them. I also decided to try "Statistics" using "table" and it gave alot better view. Good to now for the future.

I found a Blacksun textfile in the users Downloads directory.

```
index=main EventCode=11 txt TargetFilename="*"
| table TargetFilename
```

***ANSWER:*** C:\Users\keegan\Downloads\vasg6b0wmw029hd\BlackSun_README.txt

<img src="/Pseclipse/9.png" alt="description" style="max-width: 500px; width: 100%;">

### flag 10

_The script saved an image file to disk to replace the user's desktop wallpaper, which can also serve as an IOC. What is the full path of the image?_

Used the same approach as for flag 9 but instead of "txt" I tried picture extensions, "png" showed nothing but with "jpg" it showed a "blacksun.jpg"

```
index=main EventCode=11 jpg TargetFilename="*"
| table TargetFilename
```

***ANSWER:*** C:\Users\Public\Pictures\blacksun.jpg

<img src="/Pseclipse/10.png" alt="description" style="max-width: 500px; width: 100%;">

## Conclusion
This room reinforces something important: real-world security investigations aren’t cinematic. They’re methodical. They’re log-heavy. They require patience. And they reward persistence.
One thing PS Eclipse really drove home for me is just how powerful Splunk’s search engine actually is.

At first glance, it feels like a simple search bar and you can use it for it. Type something in, hit enter, get results but it can also super advanced and specific. So you can start broad and then narrow it down, but;

- Yes, the syntax can be frustrating.

- Yes, missing a field name can derail you.

- Yes, one misplaced filter can return thousands of useless events.

But it doesn't take long until you finaly understand the basics of how to structure your searches properly, it becomes incredibly efficient.

***I’ll be honest — I don’t feel “fluent” in Splunk yet.***

The search syntax still trips me up sometimes. I still occasionally return way too many results. I still have moments where I know what I want to find, but I struggle to translate that into the correct query.

But that’s exactly why this is something I will definitely keep working on in the future.

Because the more I use it, the more I see how powerful it really is.

