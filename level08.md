## Challenge
World readable files strike again. Check what that user was up to, and use it to log into flag08 account.

## Solution
A `pcap` file. Packets are always welcome at my door.
```console
level08@nebula:/home/flag08$ ls -l
total 9
-rw-r--r-- 1 root root 8302 2011-11-20 21:22 capture.pcap
```
We can use `tcpdump` to print an overview of the communication recorded in the file.
```console
level08@nebula:/home/flag08$ tcpdump -r capture.pcap | less
22:23:12.267566 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [S], seq 2635601089, win 14600, options [mss 1460,sackOK,TS val 18592800 ecr 0,nop,wscale 7], length 0
22:23:12.267694 IP 59.233.235.223.12121 > 59.233.235.218.39247: Flags [S.], seq 3131636289, ack 2635601090, win 14480, options [mss 1460,sackOK,TS val 46280417 ecr 18592800,nop,wscale 5], length 0
22:23:12.267956 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [.], ack 1, win 115, options [nop,nop,TS val 18592800 ecr 46280417], length 0
22:23:12.303574 IP 59.233.235.223.12121 > 59.233.235.218.39247: Flags [P.], seq 1:4, ack 1, win 453, options [nop,nop,TS val 46280426 ecr 18592800], length 3
22:23:12.303821 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [.], ack 4, win 115, options [nop,nop,TS val 18592804 ecr 46280426], length 0
22:23:12.303842 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 1:4, ack 4, win 115, options [nop,nop,TS val 18592804 ecr 46280426], length 3
22:23:12.303962 IP 59.233.235.223.12121 > 59.233.235.218.39247: Flags [.], ack 4, win 453, options [nop,nop,TS val 46280426 ecr 18592804], length 0
22:23:12.304147 IP 59.233.235.223.12121 > 59.233.235.218.39247: Flags [P.], seq 4:22, ack 4, win 453, options [nop,nop,TS val 46280426 ecr 18592804], length 18
22:23:12.304264 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 4:22, ack 22, win 115, options [nop,nop,TS val 18592804 ecr 46280426], length 18
22:23:12.304425 IP 59.233.235.223.12121 > 59.233.235.218.39247: Flags [P.], seq 22:46, ack 22, win 453, options [nop,nop,TS val 46280426 ecr 18592804], length 24
22:23:12.304605 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 22:89, ack 46, win 115, options [nop,nop,TS val 18592804 ecr 46280426], length 67
22:23:12.306736 IP 59.233.235.223.12121 > 59.233.235.218.39247: Flags [P.], seq 46:64, ack 89, win 453, options [nop,nop,TS val 46280427 ecr 18592804], length 18
22:23:12.306958 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 89:163, ack 64, win 115, options [nop,nop,TS val 18592804 ecr 46280427], length 74
22:23:12.307270 IP 59.233.235.223.12121 > 59.233.235.218.39247: Flags [P.], seq 64:71, ack 163, win 453, options [nop,nop,TS val 46280427 ecr 18592804], length 7
22:23:12.307408 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 163:170, ack 71, win 115, options [nop,nop,TS val 18592804 ecr 46280427], length 7
22:23:12.307704 IP 59.233.235.223.12121 > 59.233.235.218.39247: Flags [P.], seq 71:86, ack 170, win 453, options [nop,nop,TS val 46280427 ecr 18592804], length 15
22:23:12.307843 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 170:179, ack 86, win 115, options [nop,nop,TS val 18592804 ecr 46280427], length 9
22:23:12.308016 IP 59.233.235.223.12121 > 59.233.235.218.39247: Flags [P.], seq 86:127, ack 179, win 453, options [nop,nop,TS val 46280427 ecr 18592804], length 41
22:23:12.339309 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [.], ack 127, win 115, options [nop,nop,TS val 18592808 ecr 46280427], length 0
22:23:12.339391 IP 59.233.235.223.12121 > 59.233.235.218.39247: Flags [P.], seq 127:202, ack 179, win 453, options [nop,nop,TS val 46280435 ecr 18592808], length 75
22:23:12.339542 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [.], ack 202, win 115, options [nop,nop,TS val 18592808 ecr 46280435], length 0
22:23:24.491452 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 179:180, ack 202, win 115, options [nop,nop,TS val 18594023 ecr 46280435], length 1
22:23:24.496998 IP 59.233.235.223.12121 > 59.233.235.218.39247: Flags [P.], seq 202:204, ack 180, win 453, options [nop,nop,TS val 46283475 ecr 18594023], length 2
...
```
Looks like a communication between two hosts.

We can print the content of this TCP stream with `tcpflow`.
```console
level08@nebula:/home/flag08$ tcpflow -cr capture.pcap
059.233.235.223.12121-059.233.235.218.39247: ..%
059.233.235.218.39247-059.233.235.223.12121: ..%
059.233.235.223.12121-059.233.235.218.39247: ..&..... ..#..'..$
059.233.235.218.39247-059.233.235.223.12121: ..&..... ..#..'..$
059.233.235.223.12121-059.233.235.218.39247: .. .....#.....'.........
059.233.235.218.39247-059.233.235.223.12121: .. .38400,38400....#.SodaCan:0....'..DISPLAY.SodaCan:0......xterm..
059.233.235.223.12121-059.233.235.218.39247: ........"........!
059.233.235.218.39247-059.233.235.223.12121: ........"..".....b........b.....B.
..............................1.......!
059.233.235.223.12121-059.233.235.218.39247: .."....
059.233.235.218.39247-059.233.235.223.12121: .."....
059.233.235.223.12121-059.233.235.218.39247: ..!..........."
059.233.235.218.39247-059.233.235.223.12121: ........"
059.233.235.223.12121-059.233.235.218.39247: .."................
.....................
059.233.235.223.12121-059.233.235.218.39247:
Linux 2.6.38-8-generic-pae (::ffff:10.1.1.2) (pts/10)

..wwwbugs login:
059.233.235.218.39247-059.233.235.223.12121: l
059.233.235.223.12121-059.233.235.218.39247: .l
059.233.235.218.39247-059.233.235.223.12121: e
059.233.235.223.12121-059.233.235.218.39247: .e
059.233.235.218.39247-059.233.235.223.12121: v
059.233.235.223.12121-059.233.235.218.39247: .v
059.233.235.218.39247-059.233.235.223.12121: e
059.233.235.223.12121-059.233.235.218.39247: .e
059.233.235.218.39247-059.233.235.223.12121: l
059.233.235.223.12121-059.233.235.218.39247: .l
059.233.235.218.39247-059.233.235.223.12121: 8
059.233.235.223.12121-059.233.235.218.39247: .8
059.233.235.218.39247-059.233.235.223.12121:
059.233.235.223.12121-059.233.235.218.39247: .
059.233.235.223.12121-059.233.235.218.39247: .
Password:
059.233.235.218.39247-059.233.235.223.12121: b
059.233.235.218.39247-059.233.235.223.12121: a
059.233.235.218.39247-059.233.235.223.12121: c
059.233.235.218.39247-059.233.235.223.12121: k
059.233.235.218.39247-059.233.235.223.12121: d
059.233.235.218.39247-059.233.235.223.12121: o
059.233.235.218.39247-059.233.235.223.12121: o
059.233.235.218.39247-059.233.235.223.12121: r
059.233.235.218.39247-059.233.235.223.12121: .
059.233.235.218.39247-059.233.235.223.12121: .
059.233.235.218.39247-059.233.235.223.12121: .
059.233.235.218.39247-059.233.235.223.12121: 0
059.233.235.218.39247-059.233.235.223.12121: 0
059.233.235.218.39247-059.233.235.223.12121: R
059.233.235.218.39247-059.233.235.223.12121: m
059.233.235.218.39247-059.233.235.223.12121: 8
059.233.235.218.39247-059.233.235.223.12121: .
059.233.235.218.39247-059.233.235.223.12121: a
059.233.235.218.39247-059.233.235.223.12121: t
059.233.235.218.39247-059.233.235.223.12121: e
059.233.235.218.39247-059.233.235.223.12121:
059.233.235.223.12121-059.233.235.218.39247: .

059.233.235.223.12121-059.233.235.218.39247: .
059.233.235.223.12121-059.233.235.218.39247: .
Login incorrect
wwwbugs login:
```
We can identify the following:
* -> `wwwbugs login:` <- `level08`
* -> `Password:` <- `backdoor...00Rm8.ate`

Trying to login to `flag08` using `backdoor`, `00Rm8.,ate` or even `backdoor...00Rm8.ate` fails.

Could it be the `.` in password signify a non-printable character?

`tcpdump` allows printing packet data in both `hex` and `ascii`.

You can also give it a `BPF` filter.
```console
level08@nebula:/home/flag08$ tcpdump -Xr capture.pcap "tcp dst port 12121"
22:23:34.363418 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 185:186, ack 228, win 115, options [nop,nop,TS val 18595010 ecr 46283874], length 1
        0x0000:  4510 0035 a0fb 4000 4006 4a2b 3be9 ebda  E..5..@.@.J+;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 157b baa8 fb25  ;....O/Y...{...%
        0x0020:  8018 0073 96a7 0000 0101 080a 011b bcc2  ...s............
        0x0030:  02c2 3c62 62                             ..<bb
22:23:35.253053 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 186:187, ack 228, win 115, options [nop,nop,TS val 18595099 ecr 46285951], length 1
        0x0000:  4510 0035 a0fc 4000 4006 4a2a 3be9 ebda  E..5..@.@.J*;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 157c baa8 fb25  ;....O/Y...|...%
        0x0020:  8018 0073 8f30 0000 0101 080a 011b bd1b  ...s.0..........
        0x0030:  02c2 447f 61                             ..D.a
22:23:35.873401 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 187:188, ack 228, win 115, options [nop,nop,TS val 18595161 ecr 46286164], length 1
        0x0000:  4510 0035 a0fd 4000 4006 4a29 3be9 ebda  E..5..@.@.J);...
        0x0010:  3be9 ebdf 994f 2f59 9d18 157d baa8 fb25  ;....O/Y...}...%
        0x0020:  8018 0073 8c1c 0000 0101 080a 011b bd59  ...s...........Y
        0x0030:  02c2 4554 63                             ..ETc
22:23:36.343811 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 188:189, ack 228, win 115, options [nop,nop,TS val 18595208 ecr 46286319], length 1
        0x0000:  4510 0035 a0fe 4000 4006 4a28 3be9 ebda  E..5..@.@.J(;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 157e baa8 fb25  ;....O/Y...~...%
        0x0020:  8018 0073 8351 0000 0101 080a 011b bd88  ...s.Q..........
        0x0030:  02c2 45ef 6b                             ..E.k
22:23:36.573585 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 189:190, ack 228, win 115, options [nop,nop,TS val 18595231 ecr 46286436], length 1
        0x0000:  4510 0035 a0ff 4000 4006 4a27 3be9 ebda  E..5..@.@.J';...
        0x0010:  3be9 ebdf 994f 2f59 9d18 157f baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 89c4 0000 0101 080a 011b bd9f  ...s............
        0x0030:  02c2 4664 64                             ..Fdd
22:23:36.803330 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 190:191, ack 228, win 115, options [nop,nop,TS val 18595254 ecr 46286494], length 1
        0x0000:  4510 0035 a100 4000 4006 4a26 3be9 ebda  E..5..@.@.J&;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 1580 baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 7e72 0000 0101 080a 011b bdb6  ...s~r..........
        0x0030:  02c2 469e 6f                             ..F.o
22:23:36.943261 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 191:192, ack 228, win 115, options [nop,nop,TS val 18595268 ecr 46286551], length 1
        0x0000:  4510 0035 a101 4000 4006 4a25 3be9 ebda  E..5..@.@.J%;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 1581 baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 7e2a 0000 0101 080a 011b bdc4  ...s~*..........
        0x0030:  02c2 46d7 6f                             ..F.o
22:23:37.283708 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 192:193, ack 228, win 115, options [nop,nop,TS val 18595302 ecr 46286586], length 1
        0x0000:  4510 0035 a102 4000 4006 4a24 3be9 ebda  E..5..@.@.J$;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 1582 baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 7ae4 0000 0101 080a 011b bde6  ...sz...........
        0x0030:  02c2 46fa 72                             ..F.r
22:23:38.864101 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 193:194, ack 228, win 115, options [nop,nop,TS val 18595460 ecr 46286671], length 1
        0x0000:  4510 0035 a103 4000 4006 4a23 3be9 ebda  E..5..@.@.J#;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 1583 baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 6cf0 0000 0101 080a 011b be84  ...sl...........
        0x0030:  02c2 474f 7f                             ..GO.
22:23:39.233935 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 194:195, ack 228, win 115, options [nop,nop,TS val 18595497 ecr 46287066], length 1
        0x0000:  4510 0035 a104 4000 4006 4a22 3be9 ebda  E..5..@.@.J";...
        0x0010:  3be9 ebdf 994f 2f59 9d18 1584 baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 6b3f 0000 0101 080a 011b bea9  ...sk?..........
        0x0030:  02c2 48da 7f                             ..H..
22:23:39.604364 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 195:196, ack 228, win 115, options [nop,nop,TS val 18595534 ecr 46287159], length 1
        0x0000:  4510 0035 a105 4000 4006 4a21 3be9 ebda  E..5..@.@.J!;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 1585 baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 6abc 0000 0101 080a 011b bece  ...sj...........
        0x0030:  02c2 4937 7f                             ..I7.
22:23:40.374542 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 196:197, ack 228, win 115, options [nop,nop,TS val 18595611 ecr 46287251], length 1
        0x0000:  4510 0035 a106 4000 4006 4a20 3be9 ebda  E..5..@.@.J.;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 1586 baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 b912 0000 0101 080a 011b bf1b  ...s............
        0x0030:  02c2 4993 30                             ..I.0
22:23:40.574439 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 197:198, ack 228, win 115, options [nop,nop,TS val 18595631 ecr 46287444], length 1
        0x0000:  4510 0035 a107 4000 4006 4a1f 3be9 ebda  E..5..@.@.J.;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 1587 baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 b83c 0000 0101 080a 011b bf2f  ...s.<........./
        0x0030:  02c2 4a54 30                             ..JT0
22:23:42.264451 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 198:199, ack 228, win 115, options [nop,nop,TS val 18595800 ecr 46287494], length 1
        0x0000:  4510 0035 a108 4000 4006 4a1e 3be9 ebda  E..5..@.@.J.;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 1588 baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 9560 0000 0101 080a 011b bfd8  ...s.`..........
        0x0030:  02c2 4a86 52                             ..J.R
22:23:43.574954 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 199:200, ack 228, win 115, options [nop,nop,TS val 18595931 ecr 46287916], length 1
        0x0000:  4510 0035 a109 4000 4006 4a1d 3be9 ebda  E..5..@.@.J.;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 1589 baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 7836 0000 0101 080a 011b c05b  ...sx6.........[
        0x0030:  02c2 4c2c 6d                             ..L,m
22:23:44.014684 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 200:201, ack 228, win 115, options [nop,nop,TS val 18595975 ecr 46288244], length 1
        0x0000:  4510 0035 a10a 4000 4006 4a1c 3be9 ebda  E..5..@.@.J.;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 158a baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 abc1 0000 0101 080a 011b c087  ...s............
        0x0030:  02c2 4d74 38                             ..Mt8
22:23:44.635281 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 201:202, ack 228, win 115, options [nop,nop,TS val 18596037 ecr 46288354], length 1
        0x0000:  4510 0035 a10b 4000 4006 4a1b 3be9 ebda  E..5..@.@.J.;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 158b baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 6414 0000 0101 080a 011b c0c5  ...sd...........
        0x0030:  02c2 4de2 7f                             ..M..
22:23:44.805020 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 202:203, ack 228, win 115, options [nop,nop,TS val 18596054 ecr 46288509], length 1
        0x0000:  4510 0035 a10c 4000 4006 4a1a 3be9 ebda  E..5..@.@.J.;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 158c baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 8167 0000 0101 080a 011b c0d6  ...s.g..........
        0x0030:  02c2 4e7d 61                             ..N}a
22:23:45.074939 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 203:204, ack 228, win 115, options [nop,nop,TS val 18596081 ecr 46288552], length 1
        0x0000:  4510 0035 a10d 4000 4006 4a19 3be9 ebda  E..5..@.@.J.;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 158d baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 6e20 0000 0101 080a 011b c0f1  ...sn...........
        0x0030:  02c2 4ea8 74                             ..N.t
22:23:45.104894 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 204:205, ack 228, win 115, options [nop,nop,TS val 18596084 ecr 46288619], length 1
        0x0000:  4510 0035 a10e 4000 4006 4a18 3be9 ebda  E..5..@.@.J.;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 158e baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 7cd9 0000 0101 080a 011b c0f4  ...s|...........
        0x0030:  02c2 4eeb 65                             ..N.e
22:23:45.965233 IP 59.233.235.218.39247 > 59.233.235.223.12121: Flags [P.], seq 205:206, ack 228, win 115, options [nop,nop,TS val 18596170 ecr 46288626], length 1
        0x0000:  4510 0035 a10f 4000 4006 4a17 3be9 ebda  E..5..@.@.J.;...
        0x0010:  3be9 ebdf 994f 2f59 9d18 158f baa8 fb25  ;....O/Y.......%
        0x0020:  8018 0073 d47b 0000 0101 080a 011b c14a  ...s.{.........J
        0x0030:  02c2 4ef2 0d                             ..N..
```
```
ascii : b  a  c  k  d  o  o  r  .  .  .  0  0  R  m  8  .  a  t  e  .
hex   : 62 61 63 6b 64 6f 6f 72 7f 7f 7f 30 30 52 6d 38 7f 61 74 65 0d
```
Opening a simple `ascii` table reveals the meaning of the mysterious characters:
* `0x7f` - Delete (Backspace)
* `0x0d` - Carriage Return (Enter)

Thus the password should have actually been `backd00Rmate`.
```console
level08@nebula:/home/flag08$ su flag08
Password:
sh-4.2$ getflag
You have successfully executed getflag on a target account
```
