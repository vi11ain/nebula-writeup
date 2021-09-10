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
* schedule a `cron` job to run the process (`cron` is a root owned daemon)
  * `cron` executes commands using a user owned shell, thus the parent of our process will not be root `cron`.
* kill the parent process, cause child to move to a new parent - root `init` process
  * race condition, parent should be killed before child executes critical code parts
