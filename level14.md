## Challenge
This program resides in /home/flag14/flag14. It encrypts input and writes it to standard output. An encrypted token file is also in that home directory, decrypt it :)

## Solution
More reversing = more fun?
```console
level14@nebula:/home/flag14$ ls -la
total 13
drwxr-x--- 2 flag14  level14   93 2011-12-05 18:59 .
drwxr-xr-x 1 root    root     100 2012-08-27 07:18 ..
-rw-r--r-- 1 flag14  flag14   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag14  flag14  3353 2011-05-18 02:54 .bashrc
-rwsr-x--- 1 flag14  level14 7272 2011-12-05 18:59 flag14
-rw-r--r-- 1 flag14  flag14   675 2011-05-18 02:54 .profile
-rw------- 1 level14 level14   37 2011-12-05 18:59 token
```
I downloaded `flag14`, loaded it to `Ghidra` and decompiled `main()`.

Below is the result after cleaning it up from `Ghidra` mistakes, renaming `variables`, changing `numbers`' radix and adding `comments`.
```c
void main(int argc,char **argv)
{
  size_t bytes_read;
  ssize_t bytes_written;
  int e_flag_length;
  byte *input_flag;
  byte *ENCRYPT_FLAG;
  int in_GS_OFFSET;
  bool bVar1;
  bool is_flag_equals;
  int i;
  char buffer [64];
  undefined4 local_14;
  char key;
  
  local_14 = *(undefined4 *)(in_GS_OFFSET + 0x14);
  key = '\0';
  if (1 < argc) {
    bVar1 = (char **)0xfffffffb < argv;
    is_flag_equals = (byte **)(argv + 1) == (byte **)0x0;
    e_flag_length = 3;
    input_flag = (byte *)argv[1];
    ENCRYPT_FLAG = (byte *)"-e";
    do {
                    /* check if argv[1] is '-e' */
      if (e_flag_length == 0) break;
      e_flag_length = e_flag_length + -1;
      bVar1 = *input_flag < *ENCRYPT_FLAG;
      is_flag_equals = *input_flag == *ENCRYPT_FLAG;
      input_flag = input_flag + 1;
      ENCRYPT_FLAG = ENCRYPT_FLAG + 1;
    } while (is_flag_equals);
                    /* if argv[1] is '-e' start encrypting */
    if ((!bVar1 && !is_flag_equals) == bVar1) {
      do {
                    /* read 64 bytes from stdin to buffer */
        bytes_read = read(0,buffer,64);
        if ((int)bytes_read < 1) {
                    /* WARNING: Subroutine does not return */
                    /* read failed */
          exit(0);
        }
                    /* foreach byte in buffer */
        for (i = 0; i < (int)bytes_read; i = i + 1) {
                    /* add key to byte and increment key */
          buffer[i] = key + buffer[i];
          key = key + '\x01';
        }
                    /* write encrypted buffer to stdout */
        bytes_written = write(1,buffer,bytes_read);
      } while (0 < bytes_written);
                    /* WARNING: Subroutine does not return */
      exit(0);
    }
  }
                    /* print help if no arguments passed to argv */
  printf("%s\n\t-e\tEncrypt input\n",*argv);
                    /* WARNING: Subroutine does not return */
  exit(1);
}
```
And after cleaning it and leaving the interesting parts, I noticed `key` equals `i`.
```c
void main(int argc, char **argv)
{
  size_t bytes_read;
  ssize_t bytes_written;
  int i;
  char buffer[64];
  char key;

  key = '\0';
  do
  {
    /* read 64 bytes from stdin to buffer */
    bytes_read = read(0, buffer, 64);
    if ((int)bytes_read < 1)
    {
      /* read failed */
      exit(0);
    }
    /* foreach byte in buffer */
    for (i = 0; i < (int)bytes_read; i = i + 1)
    {
      /* add key to byte and increment key */
      buffer[i] = key + buffer[i];
      key = key + '\x01';
    }
    /* write encrypted buffer to stdout */
    bytes_written = write(1, buffer, bytes_read);
  } while (0 < bytes_written);

  exit(0);
}
```
Now implement an equivalent `encryption` in `Python`:
```python
def encrypt(buffer):
    buffer = bytearray(buffer)
    for i in range(len(buffer)):
        buffer[i] += i

    return bytes(buffer)
```
And after we understood `encrypt()` we can write `decrypt()`:
```python
def decrypt(buffer):
    buffer = bytearray(buffer)
    for i in reversed(range(len(buffer))):
        buffer[i] -= i

    return bytes(buffer)
```
And finally...
```console
level14@nebula:/home/flag14$ cat token
857:g67?5ABBo:BtDA?tIvLDKL{MQPSRQWW.
```
```python
In [1]: from level14 import *

In [2]: decrypt(b'857:g67?5ABBo:BtDA?tIvLDKL{MQPSRQWW.')
Out[2]: b'8457c118-887c-4e40-a5a6-33a25353165\x0b'
```
`8457c118-887c-4e40-a5a6-33a25353165`?

Looks like the `.` at the end of the `encrypted` `token` is incorrect.
```console
level14@nebula:/home/flag14$ su flag14
Password:
sh-4.2$ whoami
flag14
sh-4.2$ getflag
You have successfully executed getflag on a target account
```
