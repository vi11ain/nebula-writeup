## Challenge
The flag06 account credentials came from a legacy unix system.

## Solution
Modern linux systems save the users' encrypted passwords in `/etc/shadow`.

This file is protected by strict permissions and we can't read it.
```console
level06@nebula:/home/flag06$ ls -l /etc/shadow
-rw-r----- 1 root shadow 2540 2011-12-06 02:12 /etc/shadow
```
We can however read the file `/etc/passwd`.
```console
level06@nebula:/home/flag06$ ls -l /etc/passwd
-rw-r--r-- 1 root root 2604 2011-12-06 02:12 /etc/passwd
```
`passwd` contains user account information, but if we look carefully and read it's `man`...
```console
man 5 passwd
```
```
/etc/passwd contains one line for each user account, with seven fields
       delimited by colons (“:”). These fields are:

       ·   login name

       ·   optional encrypted password

       ·   numerical user ID

       ·   numerical group ID

       ·   user name or comment field

       ·   user home directory

       ·   optional user command interpreter

       The encrypted password field may be blank, in which case no password is
       required to authenticate as the specified login name. However, some
       applications which read the /etc/passwd file may decide not to permit any
       access at all if the password field is blank. If the password field is a
       lower-case “x”, then the encrypted password is actually stored in the
       shadow(5) file instead; there must be a corresponding line in the
       /etc/shadow file, or else the user account is invalid. If the password
       field is any other string, then it will be treated as an encrypted
       password, as specified by crypt(3).
```
... we'll notice the file contains a password field,
```console
level06@nebula:/home/flag06$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
...
```
And the user `flag06`'s field is not `x`, meaning the value is his encrypted password.
```console
level06@nebula:/home/flag06$ grep "flag06" /etc/passwd
flag06:ueqwOCnSGdsuM:993:993::/home/flag06:/bin/sh
```
Indeed encrypted passwords used to be stored in the `/etc/passwd` file (thus the file’s name).

The encryption format is `crypt` (read `man 3 crypt`), specifically `descrypt` (traditional DES).

We can surely crack it using `hashcat`.
* `-m 1500` - use hash mode `descrypt`
* `-a 3` - use attack mode `bruteforce` (no password list)
* `passwd.txt` - `flag06`'s `/etc/passwd` line
```console
> hashcat64.exe -m 1500 -a 3 passwd.txt

ueqwOCnSGdsuM:hello

Session..........: hashcat
Status...........: Cracked
Hash.Type........: descrypt, DES (Unix), Traditional DES
Hash.Target......: ueqwOCnSGdsuM
Time.Started.....: Sun May 16 20:46:07 2021 (0 secs)
Time.Estimated...: Sun May 16 20:46:07 2021 (0 secs)
Guess.Mask.......: ?1?2?2?2?2 [5]
Guess.Charset....: -1 ?l?d?u, -2 ?l?d, -3 ?l?d*!$@_, -4 Undefined
Guess.Queue......: 5/8 (62.50%)
Speed.#1.........: 27323.5 kH/s (11.13ms) @ Accel:2 Loops:1024 Thr:256 Vec:1
Recovered........: 1/1 (100.00%) Digests, 1/1 (100.00%) Salts
Progress.........: 6031360/104136192 (5.79%)
Rejected.........: 0/6031360 (0.00%)
Restore.Point....: 92160/1679616 (5.49%)
Restore.Sub.#1...: Salt:0 Amplifier:0-62 Iteration:0-1024
Candidates.#1....: sasxi -> Xhita
Hardware.Mon.#1..: Temp: 45c Fan: 24% Util: 96% Core:1898MHz Mem:3802MHz Bus:4

Started: Sun May 16 20:46:01 2021
Stopped: Sun May 16 20:46:08 2021
```
Now we can login with the cracked password `hello`.
```console
level06@nebula:/home/flag06$ su flag06
Password:
sh-4.2$ getflag
You have successfully executed getflag on a target account
```
