---
layout: post
title: "Setting up NSD3 for DNS"
date: 2013-07-12 11:31
comments: true
categories: 
- Server Administration
- DNS
---
There are two major players in the Domain Name System server market: [Bind](https://www.isc.org/downloads/bind/) and [NSD](https://www.nlnetlabs.nl/projects/nsd/). The former has been around since the 1980s and makes up a majority of the installations in use today, while the latter saw its first release around the turn of the century, and is designed to be purely authoritative (incapable of recursive DNS queries). While selecting a DNS server for [Ambrose University College](https://ambrose.edu), NSD's ability to easily run from a Chroot, and its lightweight footprint (currently using 69MB of RAM for the entire server), easily won our vote. Furthermore, NSD can make use of BIND's zonefiles, and the migration from our old server was complete in a matter of minutes.

## Securing your server

Logging in as root is a surefire way to let hackers into your box, so lets begin by creating a new user, adding them to an admin group for sudo.

```
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

You'll also want to create a private key on your computer, and disable that pesky password authentication on the server.

```
$ mkdir ~/.ssh
$ echo "ssh-rsa [your public key]" > ~/.ssh/authorized_keys
```

``` plain /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
AllowUsers spenserj
```

Reload SSH to apply the changes, and then try logging in **in a new session** to ensure everything worked. If you can't log in, you'll still have your original session to fix things up.

```
$ sudo service ssh restart
ssh stop/waiting
ssh start/running, process 1599
```

### Update the server

```
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

## Install NSD

NSD will start as soon as you install it, and complain if you don't have a config file created already, so we'll create a blank file for now.

```
$ sudo mkdir /etc/nsd3
$ sudo touch /etc/nsd3/nsd.conf
$ sudo apt-get install nsd3
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  nsd3
0 upgraded, 1 newly installed, 0 to remove and 2 not upgraded.
Need to get 931 kB of archives.
After this operation, 1,672 kB of additional disk space will be used.
Get:1 http://ca.archive.ubuntu.com/ubuntu/ precise/universe nsd3 amd64 3.2.9-1 [931 kB]
Fetched 931 kB in 1s (491 kB/s)
Selecting previously unselected package nsd3.
(Reading database ... 49711 files and directories currently installed.)
Unpacking nsd3 (from .../nsd3_3.2.9-1_amd64.deb) ...
Processing triggers for ureadahead ...
Processing triggers for man-db ...
Setting up nsd3 (3.2.9-1) ...
grep: /etc/aliases: No such file or directory
 * Building nsd3 zones... [ OK ]
 * Starting nsd3...       [ OK ]
```

### Give yourself access to `/etc/nsd3`

By default, you won't have access to /etc/nsd3 without working as root, and running as root is bad form, so let's add the nsd group to our account. We probably have some kernel updates that need to be applied, and we need to relogin for the group to apply, so this is the perfect time to reboot our server as well.

```
$ sudo usermod -a -G nsd spenserj
$ sudo reboot now
Broadcast message from spenserj@dnsmaster
    (/dev/pts/0) at 17:23 ...

The system is going down for reboot NOW!
```

# More to come shortly.
