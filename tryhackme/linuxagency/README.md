# Try Hack Me- Linux Agency CTF Walkthrough

---

## Author

**PulseEinher**

---

## Introduction

Hello, stranger — let’s begin.

Press enter or click to view image in full size

![Ready to go??](./images/linuxagency_1.webp)

---

## Challenge Link

Today’s problem is: https://tryhackme.com/room/linuxagency

---

## Challenge Overview

Machine: Linux Agency (THM)  
Path: SSH Access → User Pivoting → Enumeration → Cron Exploit → Reverse Shell → Sudo/GTFOBins Abuse → Docker Escape → CVE-2019–14287 → Root  

Key Takeaway: Chained misconfigurations — including weak user isolation, exposed cron jobs, unsafe sudo privileges, and Docker socket access — enabled step-by-step privilege escalation from a low-privileged user to full root on both container and host systems.  

Business Impact: In a real-world HITMAN-style operational environment (simulating segmented agents or roles), such weak isolation and misconfigured privilege controls would allow an attacker to pivot across internal identities, abuse automation mechanisms like cron, and ultimately escape container boundaries to compromise the underlying host — resulting in full operational takeover, exposure of sensitive mission data, and loss of control over critical infrastructure.

---

## Initial Setup

The following entry was added to the /etc/hosts file to simplify hostname-based interaction with the target system:

    <TARGET_IP> linux.thm

---

## SSH Access

As the SSH credentials were already provided, the machine was accessed using the following login credentials, and the Mission1 flag was present in the initial login verbose.

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

Note -> A direct method to get root is also listed at the end of the writeup, but if doing for learning purposes, follow the process step by step as give below.

---

## User Pivoting

The user was switched to the “mission1 ”user using these credentials:

    agent47@linuxagency:~$ su mission1
    Password: 
    mission1@linuxagency:/home/agent47$ whoami
    mission1

The Mission2 flag was captured from the “/home/mission1” directory:

    mission1@linuxagency:/home/agent47$ cd /home/mission1
    mission1@linuxagency:~$ ls
    <<MISSION2_FLAG>>

The Mission3 flag was captured from the “/home/mission2” directory:

    mission1@linuxagency:~$ su mission2
    Password: 
    mission2@linuxagency:/home/mission1$ cd /home/mission2
    mission2@linuxagency:~$ cat flag.txt 
    <<MISSION3_FLAG>>

The Mission4 flag was captured from the “/home/mission3” directory:

    mission2@linuxagency:~$ su mission3
    Password: 
    mission3@linuxagency:/home/mission2$ cd /home/mission3
    mission3@linuxagency:~$ cat flag.txt -A
    <<MISSION4_FLAG>>^MI am really sorry man the flag is stolen by some thief's.

The Mission5 flag was captured from the “/home/mission4/flag” directory:

    mission3@linuxagency:~$ su mission4
    Password: 
    mission4@linuxagency:/home/mission3$ cd /home/mission4
    mission4@linuxagency:~$ cd flag/
    mission4@linuxagency:~/flag$ cat flag.txt 
    <<MISSION5_FLAG>>

The Mission6 flag was captured from the “/home/mission5” directory:

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

The Mission7 flag was captured from the “/home/mission6/.flag” directory:

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

The Mission8 flag was captured from the “/home/mission7” directory:

    mission6@linuxagency:~/.flag$ su mission7
    Password: 
    bash: /home/mission6/.bashrc: Permission denied
    mission7@linuxagency:~/.flag$ cd /home/mission7
    mission7@linuxagency:/home/mission7$ cat flag.txt 
    <<MISSION8_FLAG>>

The Mission9 flag was captured from the “/” directory:

    mission7@linuxagency:/home/mission7$ su mission8
    Password: 
    mission8@linuxagency:/home/mission7$ cd /home/mission8
    mission8@linuxagency:~$ find / -type f -name "flag.txt" 2>/dev/null
    /flag.txt
    mission8@linuxagency:~$ cat /flag.txt 
    <<MISSION9_FLAG>>

The Mission9 flag was not located in the expected directory and was instead identified using the find utility.

The Mission10 flag was captured from the “/home/mission9” directory:

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

The Mission11 flag was captured from the “/home/mission10/folder/L4D8/L3D7/L2D2/L1D10” directory:

    mission9@linuxagency:~$ su mission10
    Password: 
    mission10@linuxagency:/home/mission9$ cd /home/mission10
    mission10@linuxagency:~$ find . -type f -name "flag.txt" 
    ./folder/L4D8/L3D7/L2D2/L1D10/flag.txt
    mission10@linuxagency:~$ cat folder/L4D8/L3D7/L2D2/L1D10/flag.txt
    <<MISSION11_FLAG>>

The Mission12 flag was captured from the “/home/mission11” directory:

    mission10@linuxagency:~$ su  mission11
    Password: 
    mission11@linuxagency:/home/mission10$ cd /home/mission11
    mission11@linuxagency:~$ cat .bashrc | grep "flag"
    export flag=$(echo fTAyN2E5Zjc2OTUzNjQ1MzcyM2NkZTZkMzNkMWE5NDRmezIxbm9pc3NpbQo= |base64 -d|rev)
    mission11@linuxagency:~$ echo $flag
    <<MISSION12_FLAG>>

The Mission12 flag was stored in a flag variable in the .bashrc file as a reverse Base64 encoded string.

The Mission13 flag was captured from the “/home/mission12” directory:

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

The Mission14 flag was captured from the “/home/mission13” directory:

    mission12@linuxagency:~$ su mission13
    Password: 
    mission13@linuxagency:/home/mission12$ cd /home/mission13
    mission13@linuxagency:~$ ls
    flag.txt
    mission13@linuxagency:~$ cat flag.txt 
    bWlzc2lvbjE0e2Q1OThkZTk1NjM5NTE0Yjk5NDE1MDc2MTdiOWU1NGQyfQo=
    mission13@linuxagency:~$ echo "bWlzc2lvbjE0e2Q1OThkZTk1NjM5NTE0Yjk5NDE1MDc2MTdiOWU1NGQyfQo=" | base64 -d
    <<MISSION14_FLAG>>

Note -> All the further decoding and decryption that was done using the online tool was done using the following tool:

Decrypt a Message - Cipher Identifier - Online Code Recognizer  
Tool to identify/recognize the type of encryption/encoding applied to a message (more 200 ciphers/codes are…  
www.dcode.fr

The Mission15 flag was captured from the “/home/mission14” directory:

    mission13@linuxagency:~$ su mission14
    Password: 
    mission14@linuxagency:/home/mission13$ cd /home/mission14
    mission14@linuxagency:~$ cat flag.txt 
    01101101011010010111001101110011011010010110111101101110001100010011010101111011011001100110001100110100001110010011000100110101011001000011100000110001001110000110001001100110011000010110010101100110011001100011000000110001001100010011100000110101011000110011001100110101001101000011011101100110001100100011010100110101001110010011011001111101

The ASCII string, when decoded using the online tool, revealed the Mission15 flag.

The Mission16 flag was captured from the “/home/mission15” directory:

    mission14@linuxagency:~$ su mission15
    Password: 
    mission15@linuxagency:/home/mission14$ cd /home/mission15
    mission15@linuxagency:~$ cat flag.txt 
    6D697373696F6E31367B38383434313764343030333363346332303931623434643763323661393038657D

The ASCII string, when decoded using the online tool, revealed the Mission16 flag.

The Mission17 flag was captured from the “/home/mission16” directory:

    mission15@linuxagency:~$ su mission16
    Password: 
    mission16@linuxagency:/home/mission15$ cd /home/mission16
    mission16@linuxagency:~$ ls
    flag
    mission16@linuxagency:~$ chmod +x flag 
    mission16@linuxagency:~$ ./flag 
    <<MISSION17_FLAG>>

Note -> All the further codes were executed using the following online compiler:

OneCompiler - Write, run and share code online | Free online compiler with 100+ languages and…  
One Compiler helps over 12.8 million users worldwide write code online.  
onecompiler.com

The Mission18 flag was captured from the “/home/mission17” directory:

    mission16@linuxagency:~$ su mission17
    Password: 
    mission17@linuxagency:/home/mission16$ cd /home/mission17
    mission17@linuxagency:~$ cat flag.java 
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

The JAVA code, when executed using the online compiler, revealed the Mission18 flag.

The Mission19 flag was captured from the “/home/mission18” directory:

    mission17@linuxagency:~$ su mission18
    Password: 
    mission18@linuxagency:/home/mission17$ cd /home/mission18
    mission18@linuxagency:~$ cat flag.rb 
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

The RUBY code, when executed using the online compiler, revealed the Mission19 flag.

The Mission20 flag was captured from the “/home/mission19” directory:

    mission18@linuxagency:~$ su mission19
    Password: 
    mission19@linuxagency:/home/mission18$ cd /home/mission19
    mission19@linuxagency:~$ cat flag.c 
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

The C code, when executed using the online compiler, revealed the Mission20 flag.

The Mission21 flag was captured from the “/home/mission20” directory:

    mission19@linuxagency:~$ su mission20
    Password: 
    mission20@linuxagency:/home/mission19$ cd /home/mission20
    mission20@linuxagency:~$ cat flag.py 
    flag = ">:  :<=ab(d76dfe2210fak1gge5e61`kgbj`bk5c0."
    for i in range(len(flag)):
        flag = (flag[:i] + chr(ord(flag[i]) ^ ord("S")) +flag[i + 1:]);
        print(flag[i], end = "");
    print()

The PYTHON code, when executed using the online compiler, revealed the Mission20 flag.

After the user was switched, a primitive shell was spawned, when the shell was upgraded using the Python utility, the Mission21 flag was revealed:

    mission20@linuxagency:~$ su mission21
    Password: 
    $ python3 -c 'import pty; pty.spawn("/bin/bash")'
    <<MISSION22_FLAG>>

The Mission23 flag was captured from the “/home/mission22” directory:

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

After the user was switched, a restricted Python shell was spawned, and it was escaped by importing a Bash shell.

The Mission24 flag was captured from the localhost service:

    mission22@linuxagency:~$ su mission23
    Password: 
    mission23@linuxagency:/home/mission22$ cd /home/mission23
    mission23@linuxagency:~$ ls
    message.txt
    mission23@linuxagency:~$ cat message.txt 
    The hosts will help you.
    [OPTIONAL] Maybe you will need curly hairs.
    mission23@linuxagency:~$ ss -tuln
    mission23@linuxagency:~$ curl 127.0.0.1:80 | grep "mission24"
    <<MISSION24_FLAG>>

The Mission25 flag was captured from the “/home/mission24” directory:

    mission23@linuxagency:~$ su mission24
    Password: 
    mission24@linuxagency:/home/mission23$ cd /home/mission24
    mission24@linuxagency:~$ export pocket=money
    mission24@linuxagency:~$ ./bribe 
    <<MISSION25_FLAG>>

The Mission26 flag was captured from the “/home/mission25” directory:

    mission24@linuxagency:~$ su mission25
    Password: 
    mission25@linuxagency:/home/mission24$ export PATH=/bin:/usr/bin:/usr/local/bin
    mission25@linuxagency:~$ cat flag.txt 
    <<MISSION26_FLAG>>

The Mission27 flag was captured from the “/home/mission26” directory:

    mission25@linuxagency:~$ su mission26
    Password: 
    mission26@linuxagency:/home/mission25$ strings flag.jpg | grep "mission27"
    <<MISSION27_FLAG>>

The Mission28 flag was captured from the “/home/mission27” directory:

    mission26@linuxagency:~$ su mission27
    Password: 
    mission27@linuxagency:/home/mission26$ cp flag.* flag.gz
    mission27@linuxagency:/home/mission26$ gunzip flag.gz
    mission27@linuxagency:/home/mission26$ strings flag | grep "mission28"
    <<MISSION28_FLAG>>

The Mission29 flag was captured from the “/home/mission28” directory:

    mission27@linuxagency:~$ su mission28
    Password: 
    mission28@linuxagency:/home/mission27$ cat txt.galf 
    <<MISSION29_FLAG>>

The Mission30 flag was captured from the “/home/mission29/bludit” directory:

    mission28@linuxagency:~$ su mission29
    Password: 
    mission29@linuxagency:/home/mission28$ cd /home/mission29/bludit
    mission29@linuxagency:~$ cat .htpasswd | grep "mission30"
    <<MISSION30_FLAG>>

The Victor flag was captured from the “/home/mission30/Escalator” directory:

    mission30@linuxagency:~/Escalator$ git log
    commit e0b807dbeb5aba190d6307f072abb60b34425d44
    Author: root <root@Xyan1d3>
    Date:   Mon Jan 11 15:36:40 2021 +0530

        Your flag is <<VICTOR_FLAG>>

---

## Cleanup

1. Lateral movement via repeated su missionX transitions should be correlated in /var/log/auth.log, and abnormal session traces across chained users should be cleaned to reduce visibility of privilege hopping.  
2. Temporary escalation artifacts such as /tmp/random.py, modified PATH/PYTHONPATH, and variables like pocket=money should be removed or reset to eliminate clear exploitation footprints.  
3. The cron-exploited script /opt/scripts/47.sh should be restored to its original state, removing any injected reverse shell payloads used during the writable window.  
4. All shells spawned via GTFOBins (zip, git, less, vim) and cron-triggered reverse connections should be terminated, ensuring no lingering elevated sessions remain active.  
5. Sensitive data accessed during escalation — like id_rsa, decoded credentials from logs, and outputs from SUID base64 abuse—should be deleted from attacker-accessible paths.  
6. Docker escape traces, including /tmp/docker usage, socket interaction, and host mount (/mnt via chroot), should be reviewed and cleaned to remove evidence of container-to-host compromise.  

---

## Remediations

1. Enforce strict privilege boundaries by removing unnecessary su access between mission users and auditing PAM/sudo configurations to prevent unrestricted lateral movement across accounts.  
2. Secure cron job execution by preventing writable script windows (e.g., /opt/scripts/47.sh), enforcing root-only ownership, and avoiding periodic overwrites that expose race-condition abuse opportunities.  
3. Harden sudo configurations by removing dangerous GTFOBins mappings (zip, git, less, vim) and restricting execution contexts to prevent shell escapes via legitimate binaries.  
4. Prevent environment-based privilege escalation by disabling SETENV where not required and restricting user-controlled variables like PYTHONPATH that enable module hijacking.  
5. Restrict access to sensitive system components such as /var/run/docker.sock, and avoid exposing Docker binaries inside containers to eliminate container escape vectors.  

---

## Conclusion

We are done with the machine……….

Let’s move to the next, till then  
Have a good day (night too)

---

## Disclaimer

This content is intended for educational purposes only.
