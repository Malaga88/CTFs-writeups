Penetration Test Report: InfosecPrep

Date: December 4, 2025 Target IP: 192.168.243.89 Author: DarkShade Malaga Assessment Type: Black Box / Boot-to-Root
1. Executive Summary

During the security assessment of the InfosecPrep host, two critical vulnerabilities were identified that resulted in a complete compromise of the system (Root Access).

The attack chain began with an Information Disclosure vulnerability on the web server, which leaked a private SSH key. This provided initial access to the system. Subsequently, a Misconfigured SUID Binary was exploited to escalate privileges from a standard user to the root superuser.

Overall Risk Rating: <span style="color:red">CRITICAL</span>
2. Technical Findings
Finding #1: Sensitive Information Disclosure (SSH Private Key)

    Severity: <span style="color:red">High</span>

    CVSS Score (Estimated): 7.5

    Description: The web server's robots.txt file, typically used to instruct search engine crawlers, contained a disallowed entry pointing to /secret.txt. Upon inspection, this file contained a Base64-encoded string which, when decoded, revealed an unencrypted OpenSSH Private Key.

    Proof of Concept:

        Enumeration of robots.txt revealed: Disallow: /secret.txt

        Retrieval of the file: curl http://<TARGET>/secret.txt

        Decoding the payload: base64 -d revealed the RSA private key.

        The key allowed SSH login as the user oscp.

    Remediation:

        Remove sensitive files (secret.txt) from the web root immediately.

        Rotate the compromised SSH keys on the server.

        Ensure sensitive administrative data is never stored in public-facing directories.

Finding #2: Privilege Escalation via SUID Misconfiguration

    Severity: <span style="color:red">Critical</span>

    CVSS Score (Estimated): 9.0

    Description: The /bin/bash binary had the SUID (Set User ID) bit enabled. This is a critical misconfiguration that allows the binary to execute with the permissions of the file owner (root), rather than the current user. This allowed a low-privileged user to spawn a root shell.

    Proof of Concept:

        Command used to identify SUID binaries: find / -perm -u=s -type f 2>/dev/null

        Output confirmed /bin/bash possessed SUID permissions.

        Exploitation command: /bin/bash -p

        Result: Access to root shell and proof.txt.

    Remediation:

        Remove the SUID bit from the bash binary immediately using the command: chmod u-s /bin/bash

        Audit all SUID binaries on the system to ensure only necessary executables (e.g., passwd, sudo) have elevated privileges.

3. Attack Narrative (Kill Chain)

    Reconnaissance: Nmap scans identified ports 22 (SSH) and 80 (HTTP).

    Enumeration: Gobuster and manual inspection of robots.txt revealed hidden artifacts.

    Initial Access: Decoded the hidden private key and gained access via SSH as user oscp.

    Privilege Escalation: Identified /bin/bash as an SUID binary. Executed it in privileged mode to gain root access.

4. Conclusion

The InfosecPrep system is currently in a highly vulnerable state. The combination of sensitive data leakage and weak internal permissions allows for trivial remote compromise. Immediate remediation of the findings listed above is required to secure the host.
