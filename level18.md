## Challenge
Analyse the C program, and look for vulnerabilities in the program. There is an easy way to solve this level, an intermediate way to solve it, and a more difficult/unreliable way to solve it.

## Solution
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
            // open passed filename (-d optarg) to global debugfile
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

```console
printf "shell" | ./flag18 -d test -vvvv
watch --interval=1 "cat test"
```

How about we set our debug log file to `/home/flag18/password`?
