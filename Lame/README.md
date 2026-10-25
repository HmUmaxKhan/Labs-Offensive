# HackTheBox — Lame (Easy) | Penetration Test Report

**Tester:** Muhammad Umar Khan
**LinkedIn:** [linkedin.com/in/hmumax15](https://linkedin.com/in/hmumax15) | **GitHub:** [github.com/HmUmaxKhan](https://github.com/HmUmaxKhan)
**Platform:** Hack The Box
**Target:** Lame (`lame.hackthebox.gr`)
**IP Address:** `10.129.116.64`
**Difficulty:** Easy
**Category:** Linux, Samba, Remote Command Execution
**Status:** In Progress (enumeration and exploit selection complete — post-exploitation evidence pending)

> **Disclaimer:** This assessment was carried out exclusively against a Hack The Box lab machine, an environment explicitly provided for authorized security training and certification practice (HTB CPTS). No unauthorized systems were accessed. This report is published for educational and portfolio purposes only.

---

## Table of Contents
1. [Executive Summary](#1-executive-summary)
2. [Scope](#2-scope)
3. [Tools Used](#3-tools-used)
4. [Methodology](#4-methodology)
5. [Reconnaissance & Enumeration](#5-reconnaissance--enumeration)
6. [Vulnerabilities Identified](#6-vulnerabilities-identified)
7. [Exploitation](#7-exploitation)
8. [Post-Exploitation](#8-post-exploitation)
9. [Risk Summary](#9-risk-summary)
10. [Remediation Summary](#10-remediation-summary)
11. [Conclusion](#11-conclusion)

---

## 1. Executive Summary

A black-box penetration test was performed against the Hack The Box target **Lame** (`10.129.116.64`) as part of CPTS (Certified Penetration Testing Specialist) practice. The assessment identified a **critical, unauthenticated remote command execution vulnerability** in the Samba service (CVE-2007-2447, "username map script"), alongside two lower-severity findings: anonymous FTP access and disabled SMB message signing.

The Samba vulnerability allows an unauthenticated attacker to execute arbitrary operating system commands on the target. Because the vulnerable `smbd` process typically runs with root privileges, successful exploitation is expected to result in complete compromise of the host without requiring a separate privilege escalation step.

## 2. Scope

| Item | Detail |
|---|---|
| Target IP | 10.129.116.64 |
| Hostname | lame.hackthebox.gr |
| Operating System | Linux (Samba 3.0.20-Debian) |
| Testing Type | Black-box, unauthenticated |
| Ports in Scope | 21, 22, 139, 445 |
| Authorization | Hack The Box lab environment |

## 3. Tools Used

- **Nmap** — port and service/version scanning
- **ftp** (CLI client) — anonymous FTP enumeration
- **Metasploit Framework** — exploit identification and execution (`exploit/multi/samba/usermap_script`)

## 4. Methodology

This engagement followed a standard penetration testing workflow:

1. Reconnaissance & Network Enumeration
2. Service Enumeration (FTP, SMB)
3. Vulnerability Identification
4. Exploitation
5. Post-Exploitation Verification
6. Reporting & Remediation

## 5. Reconnaissance & Enumeration

### 5.1 Nmap Port & Service Scan

```
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-25 01:47 -0400
Nmap scan report for 10.129.116.64
Host is up (0.22s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   Domain name: hackthebox.gr
|_  FQDN: lame.hackthebox.gr
```

**Summary of open ports:**

| Port | Service | Version | Notes |
|---|---|---|---|
| 21/tcp | FTP | vsftpd 2.3.4 | Anonymous login allowed |
| 22/tcp | SSH | OpenSSH 4.7p1 Debian | No known unauthenticated RCE for this build |
| 139/445 | SMB | Samba smbd 3.0.20-Debian | Vulnerable to CVE-2007-2447 |

### 5.2 FTP Enumeration

Anonymous authentication was accepted by the FTP service:

```
$ ftp 10.129.116.64
Connected to 10.129.116.64.
220 (vsFTPd 2.3.4)
Name (10.129.116.64:deadbot): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||34231|).
150 Here comes the directory listing.
226 Directory send OK.
```

No files or subdirectories were found in the FTP root during this assessment.

### 5.3 SMB Enumeration

Nmap identified the SMB service as **Samba smbd 3.0.20-Debian**, a version falling within the affected range (3.0.0 – 3.0.25rc3) for CVE-2007-2447.

## 6. Vulnerabilities Identified

### 6.1 [Critical] Samba "username map script" Remote Command Execution — CVE-2007-2447

| Field | Detail |
|---|---|
| Severity | Critical (unauthenticated remote command execution) |
| CWE | CWE-78 — OS Command Injection |
| Affected Component | `smbd`, Samba 3.0.0 – 3.0.25rc3 |
| Authentication Required | None (for the `SamrChangePassword` vector) |

**Description:**
Samba versions 3.0.0 through 3.0.25rc3 pass unsanitized user-supplied input (the SMB username field) directly to `/bin/sh` when the `username map script` option is configured. By embedding shell metacharacters in the username field, an attacker can inject and execute arbitrary operating system commands.

**Evidence:**
Metasploit identified a matching exploit module for the target's Samba version:

```
msf > search username map

Matching Modules
================
   #  Name                                       Disclosure Date  Rank       Check  Description
   -  ----                                       ----------------  ----       -----  -----------
   3  exploit/multi/samba/usermap_script         2007-05-14        excellent  No     Samba "username map script" Command Execution

msf > use 3
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
msf exploit(multi/samba/usermap_script) > options

Module options (exploit/multi/samba/usermap_script):
   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target host(s)
   RPORT    139              yes       The target port (TCP)

Payload options (cmd/unix/reverse_netcat):
   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   LHOST    192.168.64.21    yes       The listen address
   LPORT    4444             yes       The listen port
```

**Impact:**
Successful exploitation grants arbitrary command execution on the target. Because `smbd` commonly runs with root privileges, this typically results in an immediate root-level shell with no further privilege escalation required — representing full system compromise.

### 6.2 [Low] Anonymous FTP Login Allowed — CVE-1999-0497

| Field | Detail |
|---|---|
| Severity | Low |
| Affected Component | vsftpd 2.3.4 |

**Description:** The FTP service accepts anonymous authentication (`anonymous` / blank password), a long-standing misconfiguration class tracked under CVE-1999-0497.

**Evidence:** See Section 5.2.

**Impact:** No sensitive files were discoverable in the current FTP root during this test. However, anonymous access increases the attack surface and could expose sensitive data if the FTP root is misconfigured or changes over time.

**Note:** vsftpd 2.3.4 is also historically associated with a separate, well-known backdoor vulnerability (CVE-2011-2523). This was not tested as part of this assessment and should be verified separately if relevant to the target in question.

### 6.3 [Medium] SMB Message Signing Disabled

| Field | Detail |
|---|---|
| Severity | Medium |
| Affected Component | Samba smbd |

**Description:** Nmap's `smb-security-mode` script reports `message_signing: disabled`. Without signing, SMB traffic is susceptible to man-in-the-middle and SMB relay attacks.

**Evidence:** See Section 5.1 (Nmap host script results).

**Impact:** An attacker positioned on the network path could tamper with or relay SMB traffic.

## 7. Exploitation

1. Confirmed the target Samba version (3.0.20-Debian) via Nmap service detection.
2. Searched Metasploit for a matching module: `search username map`.
3. Selected `exploit/multi/samba/usermap_script` (`use 3`).
4. Configured module options: `RHOSTS`, `RPORT` (139), `LHOST`, `LPORT`.
5. **Next step:** execute the module (`exploit`) to trigger the vulnerable `username map script` code path and obtain a reverse shell.

> *This assessment currently documents enumeration through exploit configuration. The `exploit` command had not yet been run at the time of writing — see Section 8 for the pending verification step.*

## 8. Post-Exploitation

> **[ PLACEHOLDER — replace with your own evidence once the exploit is run ]**

- [ ] Shell obtained (paste `exploit` output / session screenshot here)
- [ ] Privilege confirmed via `id` (expected: `uid=0(root)` since `smbd` typically runs as root)
- [ ] `user.txt` located and captured (do not publish the actual flag value)
- [ ] `root.txt` located and captured (do not publish the actual flag value)

**Note on flags:** HTB flags are unique per instance and per user. For a public write-up, it is standard practice to confirm *that* a flag was captured (e.g., "user.txt and root.txt successfully retrieved") without publishing the literal flag string.

## 9. Risk Summary

| # | Finding | Severity | CVE |
|---|---|---|---|
| 1 | Samba "username map script" RCE | Critical | CVE-2007-2447 |
| 2 | SMB message signing disabled | Medium | — |
| 3 | Anonymous FTP login allowed | Low | CVE-1999-0497 |

## 10. Remediation Summary

| Finding | Recommendation | Priority |
|---|---|---|
| Samba RCE (CVE-2007-2447) | Upgrade Samba to version 3.0.25 or later (or the latest supported release). If upgrading is not immediately possible, disable the `username map script` option in `smb.conf` unless strictly required. Restrict SMB (ports 139/445) to trusted network segments via firewall rules; do not expose SMB directly to the internet. | Immediate |
| SMB message signing disabled | Enable SMB signing: set `server signing = mandatory` in `smb.conf` (or the equivalent Group Policy for Windows environments). | High |
| Anonymous FTP login allowed | Disable anonymous FTP access unless there is a specific business need. If anonymous access is required, restrict it to a read-only, non-sensitive directory and monitor access logs. Separately verify the vsftpd build is not affected by the known 2011 backdoor (CVE-2011-2523) and upgrade if needed. | Medium |
| General | Apply a consistent patch-management cadence for all internet- or network-facing services; this Samba version is over a decade past end-of-life support. | Ongoing |

## 11. Conclusion

The Lame target demonstrates a textbook example of an unauthenticated, critical-severity remote command execution vulnerability arising from unsanitized input being passed to a system shell. This engagement reinforced core CPTS skills: service version fingerprinting, CVE correlation, and Metasploit-based exploitation of legacy Samba deployments. The recommended remediations — patching, disabling unnecessary legacy options, and enforcing SMB signing — would fully address the identified risks.

---
*This report was produced as part of independent CPTS exam preparation using Hack The Box's authorized training infrastructure.*
