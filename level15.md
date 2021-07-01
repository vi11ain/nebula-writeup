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
That probably means we are dealing with a `dlopen()` that loads the `so`.

Note we are not seeing `dlopen()` in the `strace` because it isn't a `syscall` but a `library function`.
We do see `read()`, `close()` and `writeev()` which are probably what happens behind the scenes of `dlopen()`.

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
```
Now let's write a simple `so`.
```c

```

