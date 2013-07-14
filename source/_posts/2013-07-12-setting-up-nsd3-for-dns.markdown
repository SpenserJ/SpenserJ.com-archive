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

<!-- more -->

## Securing your server

Logging in as root is a surefire way to let hackers into your box, so lets begin by creating a new user, adding them to an admin group for sudo.

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

You'll also want to create a private key on your computer, and disable that pesky password authentication on the server.

``` plain 
$ mkdir ~/.ssh
$ echo "ssh-rsa [your public key]" > ~/.ssh/authorized_keys
```

``` plain /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
AllowUsers spenserj
```

Reload SSH to apply the changes, and then try logging in **in a new session** to ensure everything worked. If you can't log in, you'll still have your original session to fix things up.

``` plain 
$ sudo service ssh restart
ssh stop/waiting
ssh start/running, process 1599
```

### Update the server

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

## Install NSD

NSD will start as soon as you install it, and complain if you don't have a config file created already, so we'll create a blank file for now.

``` plain 
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

By default, you won't have access to `/etc/nsd3` without working as root, and running as root is bad form, so let's add the nsd group to our account. We probably have some kernel updates that need to be applied, and we need to relogin for the group to apply, so this is the perfect time to reboot our server as well.

``` plain 
$ sudo usermod -a -G nsd spenserj
$ sudo reboot now
Broadcast message from spenserj@dnsmaster
    (/dev/pts/0) at 17:23 ...

The system is going down for reboot NOW!
```

## Configure NSD

### Configure NSD as a Master

You'll find a sample configuration at `/etc/nsd3/nsd.conf.sample`, and [man nsd.conf](http://linux.die.net/man/5/nsd.conf) is a great place to find more information. The config is quite simplistic though, and I was un and running in minutes.

``` plain 
server:
  logfile: "/var/log/nsd.log"
  username: nsd
  hide-version: yes
  identity: "ns1.ambrose.edu"

# Primary ambrose.edu Forward Lookup Zone
zone:
  name: ambrose.edu
  zonefile: forward/ambrose.edu.zone

  notify:      dnsslave1 NOKEY
  provide-xfr: dnsslave1 NOKEY

  notify:      dnsslave2 slave2key
  provide-xfr: dnsslave2 slave2key

# Shaw Reverse Lookup Zone
zone:
  name: 237.244.66.in-addr.arpa
  zonefile: reverse/66.244.237.zone

  notify:      dnsslave1 NOKEY
  provide-xfr: dnsslave1 NOKEY

  notify:      dnsslave2 slave2key
  provide-xfr: dnsslave2 slave2key

key:
  name: slave2key
  algorithm: hmac-md5
  secret: "95hP9OTvHID2jJO2GNaeuw=="
```

You may be asking yourself "But Spenser, how do I generate that key?", and thankfully the answer is quite simple.

``` plain 
$ dd if=/dev/random bs=16 count=1 2>/dev/null | openssl base64
NzmiUkU7zA/2rQ6nnjut3w==
```

### Configure NSD as a Slave

The slave configuration is nearly identical to the master, with the only change being the `notify` and `provide-xfr` lines changing to the slave-variant. Each zone is required to have a zonefile, but you don't need to fill it out if it pulls from the master, so I just touched each file and left it empty.

``` plain 
zone:
  name: ambrose.edu
  zonefile: forward/ambrose.edu.zone

  allow-notify:      dnsmaster NOKEY
  request-xfr:  AXFR dnsmaster NOKEY
```

### Zonefiles

NSD uses BIND zonefiles, and there is a plethora of documentation and guides on how to write one, so hit up your favourite search engine and you'll be on your way. Don't forget to update the SOA Serial whenever you make a change, otherwise the next step won't work.

### Applying your configuration changes

If you've modified your `nsd.conf`, you'll need to restart NSD.

``` plain 
$ sudo nsdc restart
```

Otherwise, changes to zonefiles require the zone database to be rebuilt and reloaded.

``` plain
$ sudo nsdc rebuild
$ sudo nsdc reload
```

If you have any slaves configured, be sure to check your logs for errors. If you don't set the server verbosity to 1 (incoming notifies/transfers) or 2 (soft warnings), and don't receive an error, it is likely working.

## Confirming your Master/Slave zones

Dig is a wonderful tool when testing DNS changes. Check the forward and reverse zones of each of your nameservers, and make sure the serial and information matches. Then update the serial on your master, rebuild/reload, and check the slave again.

``` plain Check a Forward Zone
$ dig @ns2.ambrose.edu +nocmd mail.ambrose.edu any +multiline +noall +answer
mail.ambrose.edu.       10800 IN A 66.244.237.42
```

``` plain Check a Reverse Zone
$ dig @ns2.ambrose.edu +nocmd 237.244.66.in-addr.arpa. any +multiline +noall +answer
237.244.66.in-addr.arpa. 10800 IN SOA ns1.ambrose.edu. helpdesk.ambrose.edu. (
                                2013071201 ; serial
                                604800     ; refresh (1 week)
                                86400      ; retry (1 day)
                                2419200    ; expire (4 weeks)
                                604800     ; minimum (1 week)
                                )
237.244.66.in-addr.arpa. 10800 IN NS ns1.

$ dig @ns2.ambrose.edu +nocmd 42.237.244.66.in-addr.arpa. any +multiline +noall +answer
42.237.244.66.in-addr.arpa. 10800 IN PTR mail.ambrose.edu.
```

## Get a second opinion

There are quite a few automated tests that will check the responses from your nameservers, and ensure everything is in working order. I found that [Pingdom](http://dnscheck.pingdom.com/), the [Swedish Internet Infrastructure Foundation](http://dnscheck.iis.se/), and [IntoDNS](http://www.intodns.com/) were extremely helpful.
