---
layout: post
title: "Securing a Linux Server"
date: 2013-07-12 14:52
comments: true
published: false
categories: 
- Server Administration
- Security
---
It is a rare to watch someone secure a freshly installed server right off the bat, yet the world we live in makes this a necessity. So why do so many people put it off until the end, if at all? I've done the exact same thing, and it often comes down to wanting to get right into the fun stuff. Hopefully this post will show that it is far easier than you think to secure a server, and can be quite entertaining to look down from your fortress, when the attacks begin to flow.

This post is written for Ubuntu 12.04.2 LTS, however you can do similar things on any other Linux distribution.

## Where do I begin?

If this server already has a public IP, you'll want to lock down root access immediately. In fact, you'll want to lock down SSH access entirely, and make sure that only *you* can get in. Add a new user, and add it to an admin group (preconfigured in `/etc/sudoers` to have access to `sudo`).

``` plain 
$ sudo addgroup admin
Adding group 'admin' (GID 1001)
Done.

$ sudo adduser spenserj
Adding user `spenserj' ...
Adding new group `spenserj' (1002) ...
Adding new user `spenserj' (1001) with group `spenserj' ...
Creating home directory `/home/spenserj' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for spenserj
Enter the new value, or press ENTER for the default
    Full Name []: Spenser Jones
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n] y

$ sudo usermod -a -G admin spenserj
```

You’ll also want to create a private key on your computer, and disable that pesky password authentication on the server.

``` plain
$ mkdir ~/.ssh
$ echo "ssh-rsa [your public key]" > ~/.ssh/authorized_keys
```

``` plain /etc/ssh/sshd_config
PermitRootLogin no
PermitEmptyPasswords no
PasswordAuthentication no
AllowUsers spenserj
```

Reload SSH to apply the changes, and then try logging in **in a new session** to ensure everything worked. If you can't log in, you'll still have your original session to fix things up.

``` plain
$ sudo service ssh restart
ssh stop/waiting
ssh start/running, process 1599
```

## Update the server

Now that you're the only one with access to the server, you can stop worrying about a hacker sneaking in, and breathe normally again. Chances are good that there are some updates for your server, so go ahead and run those now.

``` plain
$ sudo apt-get update
...
Hit http://ca.archive.ubuntu.com precise-updates/universe Translation-en_CA
Hit http://ca.archive.ubuntu.com precise-updates/universe Translation-en
Hit http://ca.archive.ubuntu.com precise-backports/main Translation-en
Hit http://ca.archive.ubuntu.com precise-backports/multiverse Translation-en
Hit http://ca.archive.ubuntu.com precise-backports/restricted Translation-en
Hit http://ca.archive.ubuntu.com precise-backports/universe Translation-en
Fetched 3,285 kB in 5s (573 kB/s)
Reading package lists... Done

$ sudo apt-get upgrade
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages have been kept back:
  linux-headers-generic-lts-quantal linux-image-generic-lts-quantal
The following packages will be upgraded:
  accountsservice apport apt apt-transport-https apt-utils aptitude bash ...
73 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.
Need to get 61.0 MB of archives.
After this operation, 151 kB of additional disk space will be used.
Do you want to continue [Y/n]? Y
...
Setting up libisc83 (1:9.8.1.dfsg.P1-4ubuntu0.6) ...
Setting up libdns81 (1:9.8.1.dfsg.P1-4ubuntu0.6) ...
Setting up libisccc80 (1:9.8.1.dfsg.P1-4ubuntu0.6) ...
Setting up libisccfg82 (1:9.8.1.dfsg.P1-4ubuntu0.6) ...
Setting up libbind9-80 (1:9.8.1.dfsg.P1-4ubuntu0.6) ...
Setting up liblwres80 (1:9.8.1.dfsg.P1-4ubuntu0.6) ...
Setting up bind9-host (1:9.8.1.dfsg.P1-4ubuntu0.6) ...
Setting up dnsutils (1:9.8.1.dfsg.P1-4ubuntu0.6) ...
Setting up iptables (1.4.12-1ubuntu5) ...
...
```

## Install a Firewall

Running the most recent software now? Good. Go ahead and set up a firewall, and *only* allow what you need right at this moment. You can always add another exception later, and a few minutes of extra work won't kill you. IPTables comes preinstalled in Ubuntu, so go ahead and set up some rules for it.

``` plain 
$ sudo mkdir /etc/iptables
```

``` plain /etc/iptables/rules
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT DROP [0:0]
 
# Accept any related or established connections
-I INPUT  1 -m state --state RELATED,ESTABLISHED -j ACCEPT
-I OUTPUT 1 -m state --state RELATED,ESTABLISHED -j ACCEPT
 
# Allow all traffic on the loopback interface
-A INPUT  -i lo -j ACCEPT
-A OUTPUT -o lo -j ACCEPT
 
# Allow outbound DHCP request - Some hosts (Linode) automatically assign the primary IP
#-A OUTPUT -p udp --dport 67:68 --sport 67:68 -j ACCEPT

# Outbound DNS lookups
-A OUTPUT -o eth0 -p udp -m udp --dport 53 -j ACCEPT
 
# Outbound PING requests
-A OUTPUT -p icmp -j ACCEPT
 
# Outbound Network Time Protocol (NTP) request
-A OUTPUT -p udp --dport 123 --sport 123 -j ACCEPT
 
# SSH
-A INPUT  -i eth0 -p tcp -m tcp --dport 22 -m state --state NEW -j ACCEPT

COMMIT
```

Auto-load the rules with the network, and apply them now.

``` plain 
$ sudo echo "#\!/bin/sh\niptables-restore < /etc/iptables/rules" > /etc/network/if-pre-up.d/iptables
$ sudo chmod +x /etc/network/if-pre-up.d/iptables
$ sudo /etc/network/if-pre-up.d/iptables
```

## Fail2ban those wannabe-hackers

