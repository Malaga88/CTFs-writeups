# OffSec: Photographer - CTF Walkthrough

**Platform:** Proving Grounds (OffSec)  
**Difficulty:** Intermediate  
**Target IP:** 192.168.174.76  
**Date:** December 18, 2025  

## 1. Summary
Photographer is a Linux-based boot-to-root machine that emphasizes the importance of enumeration beyond standard web ports. Initial access required chaining information leaks from a misconfigured Samba share with a known vulnerability in the Koken CMS (Authenticated Arbitrary File Upload). Privilege escalation was achieved by exploiting a binary (`/usr/bin/php7.2`) with the SUID bit improperly set.

## 2. Reconnaissance

### Nmap Scan
I performed a full port scan to identify services running on non-standard ports.

```bash
nmap -p- --min-rate=1000 -sV -sC -Pn 192.168.174.76

Key Findings:

    Port 139/445: Samba (SMB) file sharing.

    Port 8000: Koken 0.22.24 (A photography CMS) running on Apache.

SMB Enumeration

I listed the available shares using smbclient and found a non-default share named sambashare.
Bash

smbclient -L //192.168.174.76 -N
smbclient //192.168.174.76/sambashare -N

I downloaded mailsent.txt, which contained an email sent to daisa@photographer.com hinting at a "secret" for the new site ("babygirl").
3. Exploitation
Initial Access (Koken CMS)

Using the credentials gathered from the SMB share (daisa@photographer.com / babygirl), I logged into the Koken admin panel on port 8000.

I exploited CVE-2015-6549 (Authenticated Arbitrary File Upload).

    I created a PHP shell (shell.php.jpg).

    I uploaded it via the "Import Content" feature.

    Using Burp Suite, I intercepted the request and renamed the file from shell.php.jpg to shell.php.

This successfully bypassed the file extension filter. I navigated to the file location in /storage/originals/ to execute the code and catch a reverse shell as www-data.
4. Privilege Escalation
SUID Enumeration

I searched for binaries with the SUID bit set:
Bash

find / -perm -u=s -type f 2>/dev/null

The output revealed /usr/bin/php7.2 was SUID root. This is a critical misconfiguration, as PHP can execute system commands.
Root Access

I used PHP's pcntl_exec function to spawn a shell. Because the binary is SUID, the spawned shell executes with root privileges.
Bash

/usr/bin/php7.2 -r "pcntl_exec('/bin/bash', ['-p']);"

Status: Rooted. System compromised.
