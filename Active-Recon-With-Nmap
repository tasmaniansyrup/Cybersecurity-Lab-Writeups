# Active Reconnaissance with Nmap and Service Interaction
**Course:** CS 456 | **Date:** September 25, 2025

---

## Skills Demonstrated
- Active network reconnaissance using Nmap
- Nmap scripting engine (NSE) for targeted service enumeration
- FTP, SMB, and HTTP service interaction
- Vulnerability identification and security analysis
- Use of Metasploitable2 as a controlled vulnerable target

---

## Overview
This lab involved performing active reconnaissance against a Metasploitable2 
container using Nmap and its scripting engine. The goal was to enumerate 
open services, interact with discovered services directly, and identify 
security vulnerabilities exposed by the target system.

---

## Environment Setup
A Metasploitable2 container was deployed locally and assigned the IP address 
`10.89.0.8`, confirmed using the `ip a` command inside the container.

---

## Nmap Scans
A series of Nmap scans were conducted against the target with increasing 
specificity:

- **Scan 1:** Comprehensive scan to identify all open ports and running services
- **Scan 2:** Credential brute-force scan targeting FTP using Nmap NSE scripts
- **Scan 3:** Targeted SMB enumeration scan over TCP port 445
- **Scan 4:** Web vulnerability scan targeting ports 80 and 8180

---

## Service Interaction

### Web Server
The HTTP service running on port 80 was accessed directly via browser at 
`http://10.89.0.8`, confirming an active web server with browsable content.

### FTP
The FTP service was first accessed using anonymous credentials 
(username: `anonymous`, password: `anonymous`), confirming that anonymous 
login was enabled.

Scan 2, which used Nmap's NSE brute-force scripts against the FTP service, 
discovered an additional valid account with username `user` and password `user`. 
These credentials were used to successfully authenticate to the FTP server, 
confirming the vulnerability.

---

## Key Discoveries

### 1. Anonymous Read/Write Access to SMB Share
Scan 3, which targeted SMB enumeration over TCP port 445, revealed that the 
`\tmp` share allows anonymous read and write access — no credentials required. 
This is a critical misconfiguration, as it allows any unauthenticated user to 
upload, modify, or delete files on the share, providing a trivial foothold for 
an attacker.

### 2. SMB User Enumeration
The same SMB scan successfully enumerated a list of user accounts present on 
the remote system using the `smb-enum-users` NSE script. Exposed usernames are 
valuable to attackers as they enable targeted dictionary or brute-force attacks 
and can help map a path toward privilege escalation depending on each account's 
access level.

### 3. Exposed Web Directories and PHP Configuration
The web vulnerability scan against ports 80 and 8180 revealed browsable file 
directories and exposed PHP configuration details. This information gives 
attackers insight into the web application's structure, technology stack, and 
potential attack surface, making it significantly easier to identify exploitable 
entry points.

### 4. SMB Message Signing Disabled
The comprehensive scan identified that SMB message signing is disabled on the 
target, flagged by Nmap as "dangerous, but default." SMB message signing 
cryptographically verifies the integrity and authenticity of SMB communications 
between client and server. Without it, all SMB traffic is vulnerable to 
man-in-the-middle attacks, where an adversary can intercept and tamper with 
messages without detection.
