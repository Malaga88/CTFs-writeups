# OnSystem: ShellDredd - CTF Walkthrough

**Platform:** Proving Grounds (OffSec)  
**Difficulty:** Intermediate  
**Target IP:** 192.168.129.130  
**Date:** December 15, 2025  

## 1. Summary
ShellDredd is a boot-to-root machine that highlights the importance of thorough port scanning and directory enumeration. The initial foothold was established by finding a hidden SSH key inside an open FTP server running on a non-standard port context. Privilege escalation was achieved by exploiting a misconfigured SUID binary (`cpulimit`) to spawn a root shell.

## 2. Reconnaissance

### Nmap Scan
I began with a full port scan to ensure I didn't miss services running on high ports.

```bash
nmap -p- --min-rate=1000 -sV -sC -Pn 192.168.129.130

Key Findings:

    Port 21: FTP (vsftpd 3.0.3) - Anonymous login allowed.

    Port 61000: SSH (OpenSSH 7.9p1) - Critical Finding: SSH running on a high port.

3. Enumeration & Initial Access
FTP Exploration

I logged into the FTP server using the anonymous credentials. A standard list command revealed an empty directory. However, knowing that standard ls commands often hide dotfiles, I dug deeper.

I attempted to blindly navigate into potential hidden directories. Through enumeration, I discovered a hidden folder named .hannah.
Bash

ftp> cd .hannah
ftp> ls -la

Inside this directory, I found an OpenSSH Private Key (id_rsa). I downloaded this to my local machine.
Gaining Access

I modified the permissions of the key to make it usable (chmod 600) and used it to authenticate as the user hannah on the non-standard SSH port found earlier.
Bash

ssh -i hannah_key -p 61000 hannah@192.168.129.130

Status: Access granted. User flag (local.txt) captured.
4. Privilege Escalation
SUID Hunting

After gaining a user shell, I checked for binaries with the SUID bit set, which allows a user to execute the file with the permissions of the file owner (Root).
Bash

find / -perm -u=s -type f 2>/dev/null

Vulnerable Binaries Found:

    /usr/bin/cpulimit

    /usr/bin/mawk

Both of these are non-standard SUID binaries. cpulimit is particularly dangerous as it can be used to spawn child processes.
Root Exploitation

I used cpulimit to spawn a shell. Since the binary runs as root, the spawned shell inherits those privileges.
Bash

/usr/bin/cpulimit -l 100 -f -- /bin/sh -p

This successfully dropped me into a root shell.

Status: Rooted. System compromised.


---
