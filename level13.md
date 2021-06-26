## Problem
There is a security check that prevents the program from continuing execution if the user invoking it does not match a specific user id.

## Solution
```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>
#include <string.h>

#define FAKEUID 1000

int main(int argc, char **argv, char **envp)
{
  int c;
  char token[256];

  if(getuid() != FAKEUID) {
      printf("Security failure detected. UID %d started us, we expect %d\n", getuid(), FAKEUID);
      printf("The system administrators will be notified of this violation\n");
      exit(EXIT_FAILURE);
  }

  // snip, sorry :)

  printf("your token is %s\n", token);
  
}
```
```console
level13@nebula:~$ ls -la /home/flag13
total 13
drwxr-x--- 2 flag13 level13   80 2011-11-20 21:22 .
drwxr-xr-x 1 root   root      60 2012-08-27 07:18 ..
-rw-r--r-- 1 flag13 flag13   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag13 flag13  3353 2011-05-18 02:54 .bashrc
-rwsr-x--- 1 flag13 level13 7321 2011-11-20 21:22 flag13
-rw-r--r-- 1 flag13 flag13   675 2011-05-18 02:54 .profile
```
Snipped part deals with the `token`, we have read access to `flag13`, can we reveal the `token`?
```console
level13@nebula:/home/flag13$ strings flag13
/lib/ld-linux.so.2
__gmon_start__
libc.so.6
_IO_stdin_used
exit
puts
__stack_chk_fail
printf
getuid
__libc_start_main
GLIBC_2.4
GLIBC_2.0
PTRhp
UWVS
[^_]
Security failure detected. UID %d started us, we expect %d
The system administrators will be notified of this violation
8mjomjh8wml;bwnh8jwbbnnwi;>;88?o;9ob
your token is %s
;*2$"(
```
`strings` doesn't reveal a `token`-ish sequence. Let's pull out the heavy guns.

Below is `main()` decompiled using `Ghidra`:
```c

void main(void)

{
  __uid_t _Var1;
  int iVar2;
  undefined4 *puVar3;
  int in_GS_OFFSET;
  byte bVar4;
  int local_118;
  undefined4 local_114;
  undefined4 local_110;
  undefined4 local_10c;
  undefined4 local_108;
  undefined4 local_104;
  undefined4 local_100;
  undefined4 local_fc;
  undefined4 local_f8;
  undefined4 local_f4;
  undefined local_f0;
  int local_14;
  
  bVar4 = 0;
  local_14 = *(int *)(in_GS_OFFSET + 0x14);
  _Var1 = getuid();
  if (_Var1 != 1000) {
    _Var1 = getuid();
    printf("Security failure detected. UID %d started us, we expect %d\n",_Var1,1000);
    puts("The system administrators will be notified of this violation");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  puVar3 = &local_114;
  for (iVar2 = 0x40; iVar2 != 0; iVar2 = iVar2 + -1) {
    *puVar3 = 0;
    puVar3 = puVar3 + (uint)bVar4 * -2 + 1;
  }
  local_114 = 0x6f6a6d38;
  local_110 = 0x38686a6d;
  local_10c = 0x3b6c6d77;
  local_108 = 0x686e7762;
  local_104 = 0x62776a38;
  local_100 = 0x776e6e62;
  local_fc = 0x3b3e3b69;
  local_f8 = 0x6f3f3838;
  local_f4 = 0x626f393b;
  local_f0 = 0;
  for (local_118 = 0; *(char *)((int)&local_114 + local_118) != '\0'; local_118 = local_118 + 1) {
    *(byte *)((int)&local_114 + local_118) = *(byte *)((int)&local_114 + local_118) ^ 0x5a;
  }
  printf("your token is %s\n",&local_114);
  if (local_14 != *(int *)(in_GS_OFFSET + 0x14)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}


```
Notice the `XOR` decryption of `local_114` using a key of `0x5a` before printing it as the token.
```c
  for (local_118 = 0; *(char *)((int)&local_114 + local_118) != '\0'; local_118 = local_118 + 1) {
    *(byte *)((int)&local_114 + local_118) = *(byte *)((int)&local_114 + local_118) ^ 0x5a;
  }
  printf("your token is %s\n",&local_114);
```
`local_114` holds the encrypted token, but what is it?

Looks like `Ghidra` misunderstood a string, as it terminates with a `\0`.
```c
  local_114 = 0x6f6a6d38;
  local_110 = 0x38686a6d;
  local_10c = 0x3b6c6d77;
  local_108 = 0x686e7762;
  local_104 = 0x62776a38;
  local_100 = 0x776e6e62;
  local_fc = 0x3b3e3b69;
  local_f8 = 0x6f3f3838;
  local_f4 = 0x626f393b;
  local_f0 = 0;
```
Going to the `assembly` instructions reveals the truth.
```asm
        08048550 8b 0a           MOV        ECX,dword ptr [EDX]=>DAT_0804874c                = 6F6A6D38h
        08048552 89 08           MOV        dword ptr [EAX]=>local_114,ECX
```
Follow `DAT_0804874c` to it's definition in `.rodata` (read-only data)
```asm
                             DAT_0804874c                                    XREF[2]:     main:08048547(*), 
                                                                                          main:08048550(R)  
        0804874c 38 6d 6a 6f     undefined4 6F6A6D38h
                             DAT_08048750                                    XREF[1]:     main:08048554(R)  
        08048750 6d 6a 68 38     undefined4 38686A6Dh
                             DAT_08048754                                    XREF[1]:     main:0804855a(R)  
        08048754 77 6d 6c 3b     undefined4 3B6C6D77h
                             DAT_08048758                                    XREF[1]:     main:08048560(R)  
        08048758 62 77 6e 68     undefined4 686E7762h
                             DAT_0804875c                                    XREF[1]:     main:08048566(R)  
        0804875c 38 6a 77 62     undefined4 62776A38h
                             DAT_08048760                                    XREF[1]:     main:0804856c(R)  
        08048760 62 6e 6e 77     undefined4 776E6E62h
                             DAT_08048764                                    XREF[1]:     main:08048572(R)  
        08048764 69 3b 3e 3b     undefined4 3B3E3B69h
                             DAT_08048768                                    XREF[1]:     main:08048578(R)  
        08048768 38 38 3f 6f     undefined4 6F3F3838h
                             DAT_0804876c                                    XREF[1]:     main:0804857e(R)  
        0804876c 3b 39 6f 62     undefined4 626F393Bh
                             DAT_08048770                                    XREF[1]:     main:08048584(R)  
        08048770 00              undefined1 00h

```
To interpret as a `string`, right-click `DAT_0804874c` -> Data -> string.
```asm
                             s_mjh8wml;bwnh8jwbbnnwi;>;88?o;9ob_08048750     XREF[2,9]:   main:08048547(*), 
                             s_wml;bwnh8jwbbnnwi;>;88?o;9ob_08048754                      main:08048550(R), 
                             s_bwnh8jwbbnnwi;>;88?o;9ob_08048758                          main:08048554(R), 
                             s_8jwbbnnwi;>;88?o;9ob_0804875c                              main:0804855a(R), 
                             s_bnnwi;>;88?o;9ob_08048760                                  main:08048560(R), 
                             s_i;>;88?o;9ob_08048764                                      main:08048566(R), 
                             s_88?o;9ob_08048768                                          main:0804856c(R), 
                             s_;9ob_0804876c                                              main:08048572(R), 
                             s__08048770                                                  main:08048578(R), 
                             s_8mjomjh8wml;bwnh8jwbbnnwi;>;88?o_0804874c                  main:0804857e(R), 
                                                                                          main:08048584(R)  
        0804874c 38 6d 6a        ds         "8mjomjh8wml;bwnh8jwbbnnwi;>;88?o;9ob"
                 6f 6d 6a 
                 68 38 77 
```
`8mjomjh8wml;bwnh8jwbbnnwi;>;88?o;9ob` looks a lot more like a `XOR` encrypted token! We can also recall it from running `strings`.

Let's decrypt it with `Python`.
```python
In [1]: token = bytearray(b'8mjomjh8wml;bwnh8jwbbnnwi;>;88?o;9ob')

In [2]: for i in range(len(token)):
   ...:     token[i] ^= 0x5a
   ...:

In [3]: token
Out[3]: bytearray(b'b705702b-76a8-42b0-8844-3adabbe5ac58')
```
Login using `b705702b-76a8-42b0-8844-3adabbe5ac58` and profit!
```console
level13@nebula:/home/flag13$ su flag13
Password:
sh-4.2$ whoami
flag13
sh-4.2$ getflag
You have successfully executed getflag on a target account
```

## Bonus Solution
There's another cool way to solve this one.
```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>
#include <string.h>

#define FAKEUID 1000

int main(int argc, char **argv, char **envp)
{
  int c;
  char token[256];

  if(getuid() != FAKEUID) {
      printf("Security failure detected. UID %d started us, we expect %d\n", getuid(), FAKEUID);
      printf("The system administrators will be notified of this violation\n");
      exit(EXIT_FAILURE);
  }

  // snip, sorry :)

  printf("your token is %s\n", token);
  
}
```
What if we could trick `getuid()` to return `1000`?

Maybe we can't trick the `syscall` function `getuid()`, but we can trick `flag13` into thinking `getuid()` is a function of our choosing.
```console
man ld.so
```
```
NAME
       ld.so/ld-linux.so - dynamic linker/loader

DESCRIPTION
       ld.so  loads  the  shared libraries needed by a program, prepares the program to run, and then runs it.
       Unless explicitly specified via the -static option to ld during compilation, all
       Linux programs are incomplete and require further linking at run time.

...
```
As described in the `man` file, `ld.so` is the `dynamic linker` responsible for loading the `shared libraries` used by a program, thus preparing it to run and then running it. We can tweak it's behaviour using a variety of `environment variables`.

A very interesting one is `LD_PRELOAD`:
> A  whitespace-separated  list  of  additional,  user-specified,  ELF shared libraries to be loaded before all others.  This can be used to selectively override functions in other
              shared libraries.  For setuid/setgid ELF binaries, only libraries in the standard search directories that are also setgid will be loaded.

Wow, so we can use it to "selectively override functions in other shared libraries"!

But before writing our overriding `shared object`, is `flag13` `dymanically linked`?
```console
level13@nebula:/home/flag13$ file flag13
flag13: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.15, not stripped
```
Yes it is, let's write an evil `getuid()`!
```console
man getuid
```
We need to implement `getuid()`'s header.
```
NAME
       getuid, geteuid - get user identity

SYNOPSIS
       #include <unistd.h>
       #include <sys/types.h>

       uid_t getuid(void);
       uid_t geteuid(void);
```
```c
#include <sys/types.h>

uid_t getuid(void){
  return 1000;
}
```
Compile it as an `so`.
```console
level13@nebula:/home/flag13$ gcc /home/level13/evil.c -shared -o /home/level13/evil.so
```
And `LD_PRELOAD`!
```console
level13@nebula:/home/flag13$ LD_PRELOAD=/home/level13/evil.so ./flag13
Security failure detected. UID 1014 started us, we expect 1000
The system administrators will be notified of this violation
```
Wait what? Oh...

Notice `flag13`'s permissions:
```console
level13@nebula:/home/flag13$ ls -la flag13
-rwsr-x--- 1 flag13 level13 7321 2011-11-20 21:22 flag13
```
`setuid` bit is set, and according to `LD_PRELOAD`'s description in `man ld.so`:
> For setuid/setgid ELF binaries, only libraries in the standard search directories that are also setgid will be loaded.

Copy the binary to remove the `setuid`.
```console
level13@nebula:/home/flag13$ cp ./flag13 /home/level13/flag13
level13@nebula:/home/flag13$ ls -la /home/level13/flag13
-rwxr-x--- 1 level13 level13 7321 2021-06-26 11:29 /home/level13/flag13
```
And run copied version.
```console
level13@nebula:/home/flag13$ LD_PRELOAD=/home/level13/evil.so /home/level13/flag13
your token is b705702b-76a8-42b0-8844-3adabbe5ac58
```
