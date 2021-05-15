## Challenge
Find a Set User ID program that will run as the “flag00” account.

## Solution
We can use the executable `find` to find such executable.
```shell
find / -user flag00 -perm /u=s | less
```
This will recursively search for files in `/` with the following properties:
* created by the user `flag00`
* with a `setuid` permission bit

```shell
/bin/.../flag00
/rofs/bin/.../flag00
```
Looks like the same file.
```shell
level00@nebula:~$ /bin/.../flag00
Congrats, now run getflag to get your flag!
flag00@nebula:~$ getflag
You have successfully executed getflag on a target account
```
