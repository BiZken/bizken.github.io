#---
layout: ../../layouts/Post.astro
title: "HackTheBox: Example Machine"
description: "Walkthrough covering initial enumeration with Nmap, exploiting SQL injection for foothold, and leveraging a SUID binary for root."
date: "2026-02-15"
type: writeup
platform: htb
tags: ["sqli", "privesc"]
backLink: /writeups
backLabel: Back to writeups
#---

## Overview

Example Machine is a medium-difficulty Linux box on HackTheBox. It involves web enumeration, SQL injection for initial access, and exploiting a misconfigured SUID binary for privilege escalation.

## Enumeration

Starting with an Nmap scan to identify open ports and services:

```
nmap -sC -sV -oN nmap/initial 10.10.10.100

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9
80/tcp open  http    Apache 2.4.52
```

Port 80 hosts a web application. Running `gobuster` reveals a login page at `/admin` and an API endpoint at `/api/users`.

## Foothold

The login page is vulnerable to SQL injection. Testing with a basic payload in the username field:

```
' OR 1=1 -- -
```

This bypasses authentication and grants access to the admin dashboard. From the dashboard, we can upload a PHP reverse shell via the file upload feature.

## Privilege Escalation

After catching the shell, checking for SUID binaries:

```bash
find / -perm -4000 -type f 2>/dev/null
```

A custom binary `/opt/backup` runs as root and accepts a filename argument without sanitisation. We can use this to read the root flag or escalate to a root shell.

> Always check SUID binaries — custom ones are often the intended privesc vector on CTF machines.

## Lessons Learned

This box reinforced the importance of thorough web enumeration and checking for common injection points. The privilege escalation was a good reminder to always audit SUID binaries on any system you're testing.
