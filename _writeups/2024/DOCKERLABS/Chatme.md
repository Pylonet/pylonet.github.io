---
layout: writeup
category: DOCKERLABS
chall_description: https://dockerlabs.es
points: 0
solves: 28427
tags: Linux FileUpload 
date: 2024-10-19
comments: true
---

# Enumeration
We will start with an nmap scan:

| PORT | SERVICE                |
|------|------------------------|
| 80   | **HTTP**: Nginx 1.24.0 |

We will be able to see that it has port 80 so we will see the web:

![](/assets/images/dockerlabs/chatme/1.webp)

We will be able to see that it is like a company that gives services of chat online, seeing the code of the Web we will be able to see a subdomain that would be in the button that we see:

![](/assets/images/dockerlabs/chatme/2.png)

We will add that to our `/etc/hosts/`:

![](/assets/images/dockerlabs/chatme/3.png)

We will now access it:

![](/assets/images/dockerlabs/chatme/4.webp)


We will be able to see a login where it only asks for a user, so I will enter one:

![](/assets/images/dockerlabs/chatme/5.webp)

We will be able to see that it is a functional chat and where a user called System has cleaned the chat, if we wait 3 minutes we will be able to see that the same user System has cleaned the chat, so it is automated for it. We will see that we can upload files and if we try to upload a .sh, .py.rb it will be uploaded to the /uploads/ folder but in .png format so it does not help us much. Remember that some time ago there was a critical vulnerability in Whatsapp where we could upload a .pyz file so I will upload a file with that extension where I do a reverse shell:

# Foothold

![](/assets/images/dockerlabs/chatme/6.png)

We will upload the file, but we will see that in the /uploads/ folder it is not found, if we wait 1 minute we will be able to see that we receive reverse shell as the system user:

![](/assets/images/dockerlabs/chatme/7.png)

# Privilege Escalation

If we do a `sudo -l` we can see that we can use procmail as root without password:

![](/assets/images/dockerlabs/chatme/8.png)

If we search on [GTFOBins](https://gtfobins.github.io/) we will not find anything, not even if we google it.

![](/assets/images/dockerlabs/chatme/9.png)

To achieve root we can create a [procmail configuration file](https://elatov.github.io/2010/02/procmail-installation-and-configuration-guide/#example-to-use-spamassassin-with-procmail), in this case we will create one where it creates a file in the `/tmp/` and we will see that it creates it as **root**:

*.procmailrc* configuration file:

![](/assets/images/dockerlabs/chatme/10.png)

![](/assets/images/dockerlabs/chatme/11.avif)

Now while doing the touch give suid to `/bin/bash`:

![](/assets/images/dockerlabs/chatme/12.png)

![](/assets/images/dockerlabs/chatme/13.png)

