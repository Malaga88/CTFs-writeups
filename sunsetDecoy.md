# Sunset: Decoy - CTF Walkthrough

**Platform:** Proving Grounds (originally VulnHub)  
**Difficulty:** Intermediate  
**Target IP:** 192.168.144.85  
**Date:** December 10, 2025  

## 1. Summary
Sunset: Decoy is a "boot-to-root" machine that emphasizes enumeration and cracking. The initial foothold requires identifying a non-standard web entry point, cracking a password-protected zip file, and chaining that information to compromise a user account. The box features a Restricted Bash (RBASH) environment that must be bypassed.

## 2. Enumeration

### Nmap Scan
I started with a full port scan to identify open services. I had to use `-Pn` as the host was blocking ICMP ping probes.

```bash
nmap -p- --min-rate=1000 -sV -Pn 192.168.144.85
Results:

    Port 22: SSH (OpenSSH 7.9p1)

    Port 80: HTTP (Apache 2.4.38)

Web Enumeration

Checking the web server on port 80 revealed a directory containing a locked zip file named save.zip.
3. Exploitation
Cracking the Zip

To access the contents of save.zip, I used zip2john to convert it into a hash format and john to crack it using the Rockyou wordlist.
Bash

zip2john save.zip > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

Password Found: manuel (or whatever the zip password was)
Extracting System Files

Inside the zip, I found a backup of the target's /etc directory, including passwd, shadow, and sudoers. This was a critical finding as it contained the user password hashes.
Cracking User Credentials

I combined the passwd and shadow files using unshadow and ran them through John the Ripper again.
Bash

unshadow etc/passwd etc/shadow > unshadowed.txt
john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt

Credentials Found:

    User: 296640a3b825115a47b68fc44501c828

    Password: server

Initial Access & RBASH Escape

I logged in via SSH but was immediately placed in a Restricted Bash (rbash) environment, preventing me from using commands like cat, cd, or ls (outside the current directory).

The Bypass: I bypassed this by forcing SSH to allocate a pseudo-terminal with a clean bash profile:
Bash

ssh 296640a3b825115a47b68fc44501c828@192.168.144.85 -t "bash --noprofile"

4. Privilege Escalation

(Document your root steps here once you finish the chkrootkit/process hunting!)

    Vulnerability Identified: ...

    Exploit Used: ...

Documented by [Your Name/Handle]


---

### How to upload this to GitHub

1.  **Create a Repo:** Go to GitHub and create a new repository called `CTF-Writeups` or `Proving-Grounds-Journey`.
2.  **Create the File:** Inside that repo, create a folder named `Sunset-Decoy` and a file inside it named `README.md`.
3.  **Paste & Commit:** Paste the template above, fill in the blanks, and commit the changes.

Would you like me to help you format the **Privilege Escalation** section once you root the box
