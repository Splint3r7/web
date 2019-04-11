---
title: "OverTheWire’s Bandit 1-17 solutions"
date: 2019-04-11
tags: [overthewire, ctf, writeup, solutions,bandit]
excerpt: "OverTheWire’s Bandit 1-17 solutions"

---

## Level 0:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit0.html

### Goal:

> The goal of this level is for you to log into the game using SSH. The host to which you need to connect is bandit.labs.overthewire.org, on port 2220. The username is bandit0 and the password is bandit0. Once logged in, go to the Level 1 page to find out how to beat Level 1.

### Solution:

```
$ ssh bandit0@bandit.labs.overthewire.org -p 2220
password:bandit0
```

## Level 0 - 1:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit1.html

### Goal

> The password for the next level is stored in a file called readme located in the home directory. Use this password to log into bandit1 using SSH. Whenever you find a password for a level, use SSH (on port 2220) to log into that level and continue the game.

### Solution:

```
$ cat readme
boJ9jbbUNNfktd78OOpsqOltutMc3MY1
```

## Level 1 - 2:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit2.html

### Goal:

> The password for the next level is stored in a file called - located in the home directory

### Solution:

```
cat < -
cat ./-
```

### Resources:
> @ @ https://unix.stackexchange.com/questions/16357/usage-of-dash-in-place-of-a-filename

## Level 2 - 3:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit3.html

### Goal:

> The password for the next level is stored in a file called spaces in this filename located in the home directory

### Solution:

```
bandit2@bandit:~$ cat "spaces in this filename"
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK
```

## Level 3 - 4:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit4.html

### Goal:

> The password for the next level is stored in a hidden file in the inhere directory.

### Solution:

```
bandit3@bandit:~/inhere$ ls -la
total 12
drwxr-xr-x 2 root    root    4096 Oct 16 14:00 .
drwxr-xr-x 3 root    root    4096 Oct 16 14:00 ..
-rw-r----- 1 bandit4 bandit3   33 Oct 16 14:00 .hidden
bandit3@bandit:~/inhere$ cat .hidden
pIwrPrtPN36QITSp3EQaw936yaFoFgAB
```

## Level 4 - 5:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit5.html

### Goal:

> The password for the next level is stored in the only human-readable file in the inhere directory. Tip: if your terminal is messed up, try the “reset” command.

### Solution:

```
bandit4@melissa:~/inhere$ file ./-*
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
bandit4@melissa:~/inhere$ cat ./-file07
koReBOKuIDDepwhWk7jZC0RTdopnAYKh
```

## Level 5 - 6:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit6.html

### Goal:

> The password for the next level is stored in a file somewhere under the inhere directory and has all of the following properties:

    > ⋅⋅* human-readable
    > ⋅⋅* 1033 bytes in size
    > ⋅⋅* not executable

### Solution:

```
bandit5@bandit:~/inhere$ find ./* -size 1033c
./maybehere07/.file2
bandit5@bandit:~/inhere$ cat < ./maybehere07/.file2
DXjZPULLxYr17uwoI01bNLQbtFemEgo7

```

## Level 6 - 7:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit7.html

### Goal:

> The password for the next level is stored somewhere on the server and has all of the following properties:

  >  ⋅⋅* owned by user bandit7
  >  ⋅⋅* owned by group bandit6
  >  ⋅⋅* 33 bytes in size


### Solution:

```
bandit6@bandit:~$ find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
bandit6@bandit:~$ cat < /var/lib/dpkg/info/bandit7.password
HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs
```

## Level 7 - 8:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit8.html

### Goal:

> The password for the next level is stored in the file data.txt next to the word millionth

### Solution:

```
bandit7@bandit:~$ cat data.txt | grep "millionth"
millionth	cvX2JJa4CFALtqS87jk27qwqGhBM9plV
```

## Level 8 - 9:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit9.html

### Goal:

> The password for the next level is stored in the file data.txt and is the only line of text that occurs only once

### Solution:

```
bandit8@bandit:~$ cat data.txt | sort |uniq -u
UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR
```

## Level 9 - 10:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit10.html

### Goal:

> The password for the next level is stored in the file data.txt in one of the few human-readable strings, beginning with several ‘=’ characters.

### Solution:

```
bandit9@bandit:~$ strings data.txt | grep "=========="
2========== the
========== password
========== isa
========== truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk
```

## Level 10 - 11:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit11.html

### Goal:

> The password for the next level is stored in the file data.txt, which contains base64 encoded data

### Solution:

```
bandit10@bandit:~# echo "VGhlIHBhc3N3b3JkIGlzIElGdWt3S0dzRlc4TU9xM0lSRnFyeEUxaHhUTkViVVBSCg==" | base64 -d
The password is IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR
```

## Level 11 - 12:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit12.html

### Goal:

> The password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions

### Solution:

```
bandit11@bandit:~$ echo "Gur cnffjbeq vf 5Gr8L4qetPEsPk8htqjhRK8XSP6x2RHh" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
The password is 5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu
```

## Level 12 - 13:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit13.html

### Goal:

> The password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work using mkdir. For example: mkdir /tmp/myname123. Then copy the datafile using cp, and rename it using mv (read the manpages!)

### Solution:

```
bandit12@bandit:/tmp/haxxan2$ xxd -r /home/bandit12/data.txt > data.txt
bandit12@bandit:/tmp/haxxan2$ zcat data.txt |file -
/dev/stdin: bzip2 compressed data, block size = 900k
bandit12@bandit:/tmp/haxxan2$ zcat data.txt |bzcat|file -
/dev/stdin: gzip compressed data, was "data4.bin", last modified: Tue Oct 16 12:00:23 2018, max compression, from Unix
bandit12@bandit:/tmp/haxxan2$ zcat data.txt |bzcat|zcat |file -
/dev/stdin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/haxxan2$ zcat data.txt |bzcat|zcat |tar x0|file -
tar: Options '-[0-7][lmh]' not supported by *this* tar
Try 'tar --help' or 'tar --usage' for more information.
/dev/stdin: empty
bandit12@bandit:/tmp/haxxan2$ zcat data.txt |bzcat|zcat|tar xO|file -
/dev/stdin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/haxxan2$ zcat data.txt |bzcat|zcat|tar xO|tar xO|file -
/dev/stdin: bzip2 compressed data, block size = 900k
bandit12@bandit:/tmp/haxxan2$ zcat data.txt |bzcat|zcat|tar xO|tar xO|bzcat |file -
/dev/stdin: POSIX tar archive (GNU)
bandit12@bandit:/tmp/haxxan2$ zcat data.txt |bzcat|zcat|tar xO|tar xO|bzcat|tar xO| file -
/dev/stdin: gzip compressed data, was "data9.bin", last modified: Tue Oct 16 12:00:23 2018, max compression, from Unix
bandit12@bandit:/tmp/haxxan2$ zcat data.txt |bzcat|zcat|tar xO|tar xO|bzcat|tar xO| zcat|file -
/dev/stdin: ASCII text
bandit12@bandit:/tmp/haxxan2$ zcat data.txt |bzcat|zcat|tar xO|tar xO|bzcat|tar xO| zcat > final.txt
bandit12@bandit:/tmp/haxxan2$ cat final.txt
The password is 8ZjyCRiBWFYkneahHwxCv3wb2a1ORpYL
```

## Level 13 - 14:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit14.html

### Goal:

> The password for the next level is stored in /etc/bandit_pass/bandit14 and can only be read by user bandit14. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. Note: localhost is a hostname that refers to the machine you are working on

### Solution:

```
bandit13@melissa:~$ ls
sshkey.private
bandit13@melissa:~$ ssh -i sshkey.private bandit14@localhost
Could not create directory '/home/bandit13/.ssh'.
The authenticity of host 'localhost (127.0.0.1)' can't be established.
RSA key fingerprint is 9d:09:d9:46:84:df:f9:dd:cc:7c:dc:49:a0:95:b2:10.
Are you sure you want to continue connecting (yes/no)? yes
Failed to add the host to the list of known hosts (/home/bandit13/.ssh/known_hosts).
bandit14@melissa:~$ cat /etc/bandit_pass/bandit14
4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e
```

## Level 14 - 15:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit15.html

### Goal:

> The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.

### Solution:

```
bandit14@bandit:~$ telnet localhost 30000
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e
Correct!
BfMYroe26WYalil77FoDi9qh59eK5xNr
```

## Level 15 - 16:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit16.html

### Goal:

> The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL encryption. Helpful note: Getting “HEARTBEATING” and “Read R BLOCK”? Use -ign_eof and read the “CONNECTED COMMANDS” section in the manpage. Next to ‘R’ and ‘Q’, the ‘B’ command also works in this version of that command…

### Solution:

```
bandit15@bandit:~$ openssl s_client -connect 127.0.0.1:30001
CONNECTED(00000003)
depth=0 CN = localhost
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = localhost
verify return:1
---
Certificate chain
 0 s:/CN=localhost
   i:/CN=localhost
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICBjCCAW+gAwIBAgIENHv1njANBgkqhkiG9w0BAQUFADAUMRIwEAYDVQQDDAls
b2NhbGhvc3QwHhcNMTkwMTEzMTkzNDMwWhcNMjAwMTEzMTkzNDMwWjAUMRIwEAYD
VQQDDAlsb2NhbGhvc3QwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBALOkCuqD
hi2X8nQ1Fu/p2hHx+3SeORNWt76H7Q/Wr8ZYapwViACu7h/d+Gn1Z4/1ZVN5Ukz3
fiS7UiA73eJg2iLo9rXzPw01hVB+IA4RtSTBmsUm7vwIgNDv5RsrYLl6JCGaZ+ns
/z5ihBHILWT3zvLyAn98HUiVAjAahiuwBtHxAgMBAAGjZTBjMBQGA1UdEQQNMAuC
CWxvY2FsaG9zdDBLBglghkgBhvhCAQ0EPhY8QXV0b21hdGljYWxseSBnZW5lcmF0
ZWQgYnkgTmNhdC4gU2VlIGh0dHBzOi8vbm1hcC5vcmcvbmNhdC8uMA0GCSqGSIb3
DQEBBQUAA4GBABiUJgikW8Ig5kuqkB3VCZiACRm5bsC4T5EH531RtzDi6Yywnqm3
xDbfWzLIoWpVq2U2pViIVsPmQ6nGjASQSlQhxsg9kZBaqYB26AcgrmbVIruywtij
JXvZkb10yrXeqtfK5bO8+kILa8XSBVbIXQ55Cn4HiHE7NtDZBOLBTl2/
-----END CERTIFICATE-----
subject=/CN=localhost
issuer=/CN=localhost
---
No client certificate CA names sent
Peer signing digest: SHA512
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 1019 bytes and written 269 bytes
Verification error: self signed certificate
---
New, TLSv1.2, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: 8E19209DF8287B470E0064737D06B563CF93EBB1F94C18962304A73EC99E33D7
    Session-ID-ctx:
    Master-Key: 0CE61D7E75D21265106AE3569EA0CCAE440372B275B01EDEBDFF38B97D90369D4A63A52C7CDDF7C087CDEA97681579F7
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - 32 a4 5b 29 4b c8 06 db-a5 7e c7 95 4f fd c4 c1   2.[)K....~..O...
    0010 - 9e dd f9 3b a1 9b 96 15-8e 22 e7 1c c7 ea 76 4b   ...;....."....vK
    0020 - 91 c3 c4 7d 29 92 0d 15-31 4b 99 22 48 f4 24 9f   ...})...1K."H.$.
    0030 - 05 5f 77 4e 7f dc cf d4-5c c1 8c 04 03 c7 22 07   ._wN....\.....".
    0040 - 87 db 41 c0 be 2e 66 b8-a1 c8 c2 50 96 62 a1 69   ..A...f....P.b.i
    0050 - d4 48 9e c8 b5 d6 4a 8f-a1 79 c2 0d d9 eb db b6   .H....J..y......
    0060 - 7a 71 30 cf 57 3c e9 7c-d2 89 71 be 1e b3 ab e0   zq0.W<.|..q.....
    0070 - 07 77 23 b7 8a fc cf d6-e4 30 c7 0f 5f a6 ff d2   .w#......0.._...
    0080 - ab 7b 77 8b 6b be 89 c5-13 5b 35 5c 63 8f 04 28   .{w.k....[5\c..(
    0090 - be 3e 15 f8 d8 1e e6 2d-16 2b b3 92 99 28 58 64   .>.....-.+...(Xd

    Start Time: 1547944552
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
    Extended master secret: yes
---
BfMYroe26WYalil77FoDi9qh59eK5xNr
Correct!
cluFn7wTiGryunymYOu4RcffSxQluehd
```

## Level 16 - 17:

> Challenge URL: http://overthewire.org/wargames/bandit/bandit17.html

### Goal:

> The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

### Solution:

```
bandit16@bandit:~$ nmap -p 31000-32000 127.0.0.1

Starting Nmap 7.40 ( https://nmap.org ) at 2019-01-20 01:49 CET
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00019s latency).
Not shown: 998 closed ports
PORT      STATE SERVICE
31518/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds


bandit16@bandit:~$ openssl s_client -connect 127.0.0.1:31790
CONNECTED(00000003)
depth=0 CN = localhost
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = localhost
verify return:1
---
Certificate chain
 0 s:/CN=localhost
   i:/CN=localhost
---
Server certificate
-----BEGIN CERTIFICATE-----
MIICBjCCAW+gAwIBAgIELHlSDDANBgkqhkiG9w0BAQUFADAUMRIwEAYDVQQDDAls
b2NhbGhvc3QwHhcNMTkwMTEzMTkzNDMwWhcNMjAwMTEzMTkzNDMwWjAUMRIwEAYD
VQQDDAlsb2NhbGhvc3QwgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAJ4UdWRN
ZmKD+47wp9PM6kCsCLM6u3WD1ByGlnJn3HeILlCxId2oaklGkOx/BzYcX8m0U+wU
XrC8/lC0o1YHfgfCaVht4ubhmrlD4iUnY7pABtIsrIdLgz/Ee54121GG2+ZZfmpq
mMytFoWyHSQUNFtavUMPYkzNCI8FO7GiLIZrAgMBAAGjZTBjMBQGA1UdEQQNMAuC
CWxvY2FsaG9zdDBLBglghkgBhvhCAQ0EPhY8QXV0b21hdGljYWxseSBnZW5lcmF0
ZWQgYnkgTmNhdC4gU2VlIGh0dHBzOi8vbm1hcC5vcmcvbmNhdC8uMA0GCSqGSIb3
DQEBBQUAA4GBAEtU/dx2IFDG7q3IcOmYkrvzHlmUKMXPiQqJgim/nzAo89is3A2B
S9+oWaaCX+Z5Hz97h7nJNFQHTjpoKKP0wS7ZqdpXFXFOVa4rD12hPEebqE/fCPiC
bOoJkI78ZtUg3fbal8XPHyQK4QcOfgfVAIjSgoUfiNJYQz+0Yqh6EMEp
-----END CERTIFICATE-----
subject=/CN=localhost
issuer=/CN=localhost
---
No client certificate CA names sent
Peer signing digest: SHA512
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 1019 bytes and written 269 bytes
Verification error: self signed certificate
---
New, TLSv1.2, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: DA335E7171F11DDF738E87F8D366DAA44D5E1DE3350AD5E9CBFAB09D1FC61D50
    Session-ID-ctx:
    Master-Key: D8F29304493DF77359CFA5C47CCEFE039CB1307EE7BA8F7FE3A32F537E52C352F15AB11C5669DB3949050C2AFB89B184
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - fe 2e da f4 cd cd e9 23-46 73 02 6e ca e6 93 3a   .......#Fs.n...:
    0010 - 14 b9 1a c9 99 c7 45 f7-57 da 64 77 ec 2f 0f b4   ......E.W.dw./..
    0020 - c5 83 44 5c b7 50 9b cb-57 00 85 ba 29 5b d7 ff   ..D\.P..W...)[..
    0030 - 19 1a 8d fa 6f 8f 95 58-75 e6 67 37 56 4e 1a d1   ....o..Xu.g7VN..
    0040 - fa 1c 3d eb c6 7b fb 2f-3e 34 3b a5 89 89 07 90   ..=..{./>4;.....
    0050 - 7d 49 d3 5f 25 59 94 a8-c4 45 71 08 42 2a 55 a7   }I._%Y...Eq.B*U.
    0060 - 9b 5d ee be 12 1c 8c 31-75 91 23 99 25 51 75 31   .].....1u.#.%Qu1
    0070 - fe 07 ec 8a 4d b5 a4 48-a6 36 08 8f f0 08 af 79   ....M..H.6.....y
    0080 - 30 68 5b 6b 79 b7 1d 47-a1 61 8c 3e be 20 2f a1   0h[ky..G.a.>. /.
    0090 - 4d 66 07 ed 61 3d 90 a7-33 fa 05 72 a2 6a 5c 27   Mf..a=..3..r.j\'

    Start Time: 1547945412
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
    Extended master secret: yes
---
cluFn7wTiGryunymYOu4RcffSxQluehd
Correct!
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvmOkuifmMg6HL2YPIOjon6iWfbp7c3jx34YkYWqUH57SUdyJ
imZzeyGC0gtZPGujUSxiJSWI/oTqexh+cAMTSMlOJf7+BrJObArnxd9Y7YT2bRPQ
Ja6Lzb558YW3FZl87ORiO+rW4LCDCNd2lUvLE/GL2GWyuKN0K5iCd5TbtJzEkQTu
DSt2mcNn4rhAL+JFr56o4T6z8WWAW18BR6yGrMq7Q/kALHYW3OekePQAzL0VUYbW
JGTi65CxbCnzc/w4+mqQyvmzpWtMAzJTzAzQxNbkR2MBGySxDLrjg0LWN6sK7wNX
x0YVztz/zbIkPjfkU1jHS+9EbVNj+D1XFOJuaQIDAQABAoIBABagpxpM1aoLWfvD
KHcj10nqcoBc4oE11aFYQwik7xfW+24pRNuDE6SFthOar69jp5RlLwD1NhPx3iBl
J9nOM8OJ0VToum43UOS8YxF8WwhXriYGnc1sskbwpXOUDc9uX4+UESzH22P29ovd
d8WErY0gPxun8pbJLmxkAtWNhpMvfe0050vk9TL5wqbu9AlbssgTcCXkMQnPw9nC
YNN6DDP2lbcBrvgT9YCNL6C+ZKufD52yOQ9qOkwFTEQpjtF4uNtJom+asvlpmS8A
vLY9r60wYSvmZhNqBUrj7lyCtXMIu1kkd4w7F77k+DjHoAXyxcUp1DGL51sOmama
+TOWWgECgYEA8JtPxP0GRJ+IQkX262jM3dEIkza8ky5moIwUqYdsx0NxHgRRhORT
8c8hAuRBb2G82so8vUHk/fur85OEfc9TncnCY2crpoqsghifKLxrLgtT+qDpfZnx
SatLdt8GfQ85yA7hnWWJ2MxF3NaeSDm75Lsm+tBbAiyc9P2jGRNtMSkCgYEAypHd
HCctNi/FwjulhttFx/rHYKhLidZDFYeiE/v45bN4yFm8x7R/b0iE7KaszX+Exdvt
SghaTdcG0Knyw1bpJVyusavPzpaJMjdJ6tcFhVAbAjm7enCIvGCSx+X3l5SiWg0A
R57hJglezIiVjv3aGwHwvlZvtszK6zV6oXFAu0ECgYAbjo46T4hyP5tJi93V5HDi
Ttiek7xRVxUl+iU7rWkGAXFpMLFteQEsRr7PJ/lemmEY5eTDAFMLy9FL2m9oQWCg
R8VdwSk8r9FGLS+9aKcV5PI/WEKlwgXinB3OhYimtiG2Cg5JCqIZFHxD6MjEGOiu
L8ktHMPvodBwNsSBULpG0QKBgBAplTfC1HOnWiMGOU3KPwYWt0O6CdTkmJOmL8Ni
blh9elyZ9FsGxsgtRBXRsqXuz7wtsQAgLHxbdLq/ZJQ7YfzOKU4ZxEnabvXnvWkU
YOdjHdSOoKvDQNWu6ucyLRAWFuISeXw9a/9p7ftpxm0TSgyvmfLF2MIAEwyzRqaM
77pBAoGAMmjmIJdjp+Ez8duyn3ieo36yrttF5NSsJLAbxFpdlc1gvtGCWW+9Cq0b
dxviW8+TFVEBl1O4f7HVm6EpTscdDxU+bCXWkfjuRb7Dy9GOtt9JPsX8MBTakzh3
vBgsyi/sN3RqRBcGU40fOoZyfAMT8s1m/uYv52O6IgeuZ/ujbjY=

1- Save the above RSA KEY into file //ssh-Key
2- set permissions to 600
$ chmod 600 ssh-key
Now login to next level using the key.
$ ssh -i ssh-key bandit17@bandit.labs.overthewire.org
done!
```
