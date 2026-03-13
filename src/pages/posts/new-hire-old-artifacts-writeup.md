
---
layout: ../../layouts/Post.astro
title: "New Hire Old Artifacts"
description: "Looking through SYSMON and windows logs with Splunk to identify malicious tools"
type: writeup
platform: THM
tags: ["SIEM", "Forensics", "Windows", "Splunk"]
backLink: /writeups
backLabel: Back to writeups
---


# New Hire Old Artifacts — TryHackMe Writeup

**Platform:** TryHackMe  
**Category:** SOC / SIEM / Splunk  
**Difficulty:** Medium  
**Tools Used:** Splunk, VirusTotal  

---

## Scenario

A new employee at WidgetLLC has been flagged for suspicious activity on their corporate workstation (DESKTOP-H1ATIJC). As a SOC analyst, the task is to investigate Sysmon and Windows event logs ingested into Splunk to determine what happened, identify malicious tools and techniques, and trace indicators of compromise.

The investigation focuses on user **FINANC-1** (and associated account **Finance01**) during **December 2021**.

---

## Flags

### Flag 1:
_A Web Browser Password Viewer executed on the infected machine. What is the name of the binary? Enter the full path_ 
***ANSWER:*** _C:\Users\FINANC~1\AppData\Local\Temp\11111.exe_

The first step was to search for any triggered Sigma-style detection rules across all user-space processes. The Splunk query:

```
index=main RuleName="*", Image=* "Users"
| table RuleName, Image
| uniq
```

The path `C:\Users\FINANC-1\AppData\Local\Temp\11111.exe` stood out immediately as an obvious attempt to disguise a tool under a generic numeric filename.

<img src="/NEWHIRE/1.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/NEWHIRE/1_1.png" alt="Res1" style="max-width: 500px; width: 100%;">



### Flag 2:
_What is listed as the company name?_
***ANSWER:*** _NirSoft_

Drilling into the Sysmon event for `11111.exe` revealed:

| Field | Value |
|---|---|
| **Company** | NirSoft |
| **Description** | Web Browser Password Viewer |
| **FileVersion** | 2.06 |
| **CommandLine** | `11111.exe /stab C:\Users\FINANC~1\AppData\Local\Temp\f4ghga23_fsa.txt` |
| **CurrentDirectory** | `C:\Users\Finance01\Pictures\Adobe Films\` |
| **IntegrityLevel** | High |

This is a known NirSoft utility (**WebBrowserPassView**) used to extract saved passwords from web browsers. The `/stab` flag writes output in tab-delimited format to a file — a classic credential harvesting move. The process ran at **High** integrity, meaning UAC was bypassed or the user had elevated privileges.
<img src="/NEWHIRE/2.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Flag 3:
_Another suspicious binary running from the same folder was executed on the workstation. What was the name of the binary? What is listed as its original filename?_
***ANSWER:*** IonicLarge.exe,PalitExplorer.exe

Searching for all files written to the FINANC-1 temp directory:

```
index=main TargetFilename="C:\\Users\\FINANC-1\\AppData\\Local\\Temp\\*"
| table TargetFilename
| uniq
```

This returned **899 events** across **847 unique files** (43 pages), including:

- `a.txt`
- `scandinavians.dat`
- `11111.exe`
- `IonicLarge.exe`
- `Procmon64.exe`


<img src="/NEWHIRE/3.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/NEWHIRE/3_2.png" alt="Res1" style="max-width: 500px; width: 100%;">


### Flag 4
_The binary from the previous question made two outbound connections to a malicious IP address. What was the IP address? Enter the answer in a defang format._

***Answer:*** _2[.]56[.]59[.]42_




Searching for outbound connections from `IonicLarge.exe`:

```
index=main IonicLarge.exe DestinationIp="*"
| table DestinationIp
| uniq
```

This returned **87 events** across **11 unique destination IPs**, including:

- `2.56.59.42`
- `34.117.59.81`
- `212.193.30.45`
- `148.251.234.93`
- `142.250.191.132`
- `104.27.40.48`

<img src="/NEWHIRE/4_1.png" alt="Res1" style="max-width: 500px; width: 100%;">


**VirusTotal lookup on 2.56.59.42** confirmed it as malicious: **13/94** security vendors flagged it, with detections including Malware, Phishing, and Malicious across vendors like Kaspersky, Fortinet, BitDefender, G-Data, and others. The IP belongs to **AS 3758 (SingNet)** in **Singapore**.

<img src="/NEWHIRE/4.png" alt="Res1" style="max-width: 500px; width: 100%;">


### Flag 5
_The same binary made some change to a registry key. What was the key path?_
***Answer:*** _HKLM\SOFTWARE\Policies\Microsoft\Windows Defender_

Searching for registry modifications made by IonicLarge.exe:

```
index=main IonicLarge.exe TaskCategory="Registry value set (rule: RegistryEvent)"
```

This returned **9 events**, one of which showed the process modifying:

```
HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection\DisableRawWriteNotification
```

This is a clear **defense evasion** technique (MITRE T1562.001: Disable or Modify Tools). The attacker tampered with Windows Defender's real-time protection settings to avoid detection.

<img src="/NEWHIRE/5.png" alt="Res1" style="max-width: 500px; width: 100%;">


### Flag 6
_Some processes were killed and the associated binaries were deleted. What were the names of the two binaries?_
***Answer:*** _WvmIOrcfsuILdX6SNwIRmGOJ.exe,phcIAmLJMAIMSa9j9MpgJo1m.exe_

Investigating use of `taskkill.exe`:

```
index=main OriginalFileName="taskkill.exe" ParentCommandLine="*"
| table ParentCommandLine
```

Two events were found:

1. `cmd.exe /c taskkill /im "WvmIOrcfsuILdk6SNwIRmGOJ.exe" /f & erase "C:\Users\Finance01\Pictures\Adobe Films\WvmIOrcfsuILdk6SNwIRmGOJ.exe" & exit`
2. `cmd.exe /c taskkill /im "phcIAmLJMAIMSa9j9MpgJoIm.exe" /f & timeout /t 6 & del /f /q "C:\Users\Finance01\Pictures\Adobe Films\phcIAmLJMAIMSa9j9MpgJoIm.exe" & del C:\ProgramData\*.dll & exit`

The attacker killed their own processes and deleted the binaries and associated DLLs to cover their tracks. The second command also includes a `timeout /t 6` delay before cleanup and deletes all DLLs from `C:\ProgramData\` — a broad anti-forensics sweep.

<img src="/NEWHIRE/6.png" alt="Res1" style="max-width: 500px; width: 100%;">


### Flag 7
_The attacker ran several commands within a PowerShell session to change the behaviour of Windows Defender. What was the last command executed in the series of similar commands?_
***Answer:*** _powershell WMIC /NAMESPACE:\\root\Microsoft\Windows\Defender PATH MSFT_MpPreference call Add ThreatIDDefaultAction_Ids=2147737394 ThreatIDDefaultAction_Actions=6 Force=True_


Searching for PowerShell-based Defender manipulation:

```
index=main powershell Image="C:\\Windows\\SysWOW64\\WindowsPowerShell\\v1.0\\powershell.exe" defender CommandLine="*"
| table CommandLine
| sort -_time
```

Four events were found, all executing WMIC commands to add **ThreatIDDefaultAction** entries to Windows Defender:

| ThreatID | Action |
|---|---|
| 2147737007 | Allow (Action=6) |
| 2147737010 | Allow (Action=6) |
| 2147735503 | Allow (Action=6) |
| 2147737394 | Allow (Action=6) |

The command pattern:

```
powershell WMIC /NAMESPACE:\\root\Microsoft\Windows\Defender PATH MSFT_MpPreference call Add ThreatIDDefaultAction_Ids=<ID> ThreatIDDefaultAction_Actions=6 Force=True
```

Action **6** corresponds to **Allow**, meaning the attacker whitelisted specific threat detections in Defender so their tools would not be quarantined. This is another instance of **T1562.001 — Disable or Modify Tools**.

<img src="/NEWHIRE/7.png" alt="Res1" style="max-width: 500px; width: 100%;">


### Flag 8
_Based on the previous answer, what were the four IDs set by the attacker? Enter the answer in order of execution. (format: 1st,2nd,3rd,4th)_
***Answer:*** _2147735503,2147737010,2147737007,2147737394_

<img src="/NEWHIRE/8.png" alt="Res1" style="max-width: 500px; width: 100%;">


### Flag 9
_Another malicious binary was executed on the infected workstation from another AppData location. What was the full path to the binary?_
***Answer:*** 

Broadening the search for executables running from AppData:

```
index=main "C:\\Users\\FINANC-1\\APPDATA\\*" exe
| table Image
| uniq
```

This returned **5,194 events** across **224 unique images**, including:

```
C:\Users\Finance01\AppData\Roaming\EasyCalc\EasyCalc.exe
```
<img src="/NEWHIRE/9.png" alt="Res1" style="max-width: 500px; width: 100%;">

Investigating the DLLs loaded by EasyCalc.exe:

```
index=main Image="C:\Users\Finance01\AppData\Roaming\EasyCalc\EasyCalc.exe"
| table ImageLoaded
| uniq
```

This returned **1,125 events** across **87 unique DLLs**, including:

- `ffmpeg.dll`
- `nw.dll`
- `nw_elf.dll`


### Flag 10
_What were the DLLs that were loaded from the binary from the previous question?_
***Answer:*** _ffmpeg.dll,nw.dll,nw_elf.dll_

<img src="/NEWHIRE/10.png" alt="Res1" style="max-width: 500px; width: 100%;">


---
