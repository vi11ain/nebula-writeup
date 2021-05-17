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
