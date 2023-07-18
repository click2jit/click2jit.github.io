---
title: Tryhackme | Dailybugle
date: 2023-07-18 09:24:00 +0530
categories: [Injection]
tags: [joomla, yum]     # TAG names should always be lowercase
---

Tryhackme Room: [https://tryhackme.com/room/dailybugle](https://tryhackme.com/room/dailybugle)

## Nmap Scan:

![a852286dc41e63dc9928fbaf554c083e.png](/_resources/a852286dc41e63dc9928fbaf554c083e.png)

**Open Ports:**

- 22 ssh
- 80 http
- 3306 mysql

**Port 80 (HTTP)**
It opens the following page:
![9ae166b729ae75b299d40772970b9099.png](/_resources/9ae166b729ae75b299d40772970b9099.png)

First we tried to login to the Login form with random default password, but it failed. We also checked the source page but nothing interesting came out.

Next we checked the robots.txt file. We got some interesting information as follows:

![26687dde940a0d88bdf304b08bddeb3b.png](/_resources/26687dde940a0d88bdf304b08bddeb3b.png)

It seems like joomla is installed. So we will look for it.

We found joomla at /administrator.

![c54faa7379577ac1f9946d0233d1f6b8.png](/_resources/c54faa7379577ac1f9946d0233d1f6b8.png)

We have searched the page for the version, but we didn't found. So we searched online and found that we can check for version from the following link:

```
    http://www.[thejoomlawebsite].com/administrator/manifests/files/joomla.xml
```

We tried on our victim machine and found the version:

![a2558691c45a95c8e7dc4e37c7b97d7c.png](/_resources/a2558691c45a95c8e7dc4e37c7b97d7c.png)

Next we searched for exploit and found the following python script from github:
https://github.com/stefanlucas/Exploit-Joomla/blob/master/joomblah.py

Downloaded to our machine and run the script. We found the User and password hash:

![8330be31795c75015e0979033b907913.png](/_resources/8330be31795c75015e0979033b907913.png)

Found from the hashcat Wiki, the hash type is bcrypt:

![c28a47ac841933832fe1f7e99230c3d4.png](/_resources/c28a47ac841933832fe1f7e99230c3d4.png)

So, next we pass the hash to hashcat and it cracked for us:

![d2c8a23cad4248e6bc8b80735b8dbb21.png](/_resources/d2c8a23cad4248e6bc8b80735b8dbb21.png)

Next, with the credentials we will log into the joomla administrator. The Control Panel will look like this:

![3976c8a7c66e65fdd968627625ceb35c.png](/_resources/3976c8a7c66e65fdd968627625ceb35c.png)

Then, we searched for any upload function for getting our reverse shell and found on the template section.

![abf0de654735af59ee2bdf2578f9354c.png](/_resources/abf0de654735af59ee2bdf2578f9354c.png)

Accessed this following path and got the reverse shell:

```
    http://10.10.247.226/templates/protostar/shell.php
```

![49cda998af13f2fed7a82607ad479b59.png](/_resources/49cda998af13f2fed7a82607ad479b59.png)

Next, we searched for any sensitive information files, and we got it inside /var/www/html file:

![d1b0f1d001f481cf9f8136dabffb5c91.png](/_resources/d1b0f1d001f481cf9f8136dabffb5c91.png)

Tried to ssh to root with this password but failed. Then tried to login to other user in the machine.
And finally succeeded to ssh into jjameson account with that password:

![8866359b5afc47f8611b035ded3070cc.png](/_resources/8866359b5afc47f8611b035ded3070cc.png)

## Privilege Escalation

Next, we checked sudo -l and we saw that we can run yum binary without password:

![ecec9582e47c3345f96242168a0d530f.png](/_resources/ecec9582e47c3345f96242168a0d530f.png)

So, we grab the sudo privilege escalation commands from gtfobins:

```bash
    TF=$(mktemp -d)
    cat >$TF/x<<EOF
    [main]
    plugins=1
    pluginpath=$TF
    pluginconfpath=$TF
    EOF

    cat >$TF/y.conf<<EOF
    [main]
    enabled=1
    EOF

    cat >$TF/y.py<<EOF
    import os
    import yum
    from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
    requires_api_version='2.1'
    def init_hook(conduit):
      os.execl('/bin/sh','/bin/sh')
    EOF

    sudo yum -c $TF/x --enableplugin=y
```

![f0dc07b55ba039429f28aaf93f071b96.png](/_resources/f0dc07b55ba039429f28aaf93f071b96.png)

Finally, we rooted this machine from Tryhackme. It was a challenging and fun machine to solve.