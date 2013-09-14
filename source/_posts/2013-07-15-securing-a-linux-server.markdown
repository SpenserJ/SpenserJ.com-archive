---
layout: post
title: "Securing a Linux Server"
date: 2013-07-15 10:52
comments: true
categories: 
- Server Administration
- Security
description: "Security should be your first concern after installing a new server, and a few simple steps can have your new box locked down in minutes."
keywords: Security, Server Administration, Linux, Ubuntu
---
It is a rarity to watch someone secure a freshly installed server right off the bat, yet the world we live in makes this a necessity. So why do so many people put it off until the end, if at all? I've done the exact same thing, and it often comes down to wanting to get right into the fun stuff. Hopefully this post will show that it is far easier than you think to secure a server, and can be quite entertaining to look down from your fortress, when the attacks begin to flow.

This post is written for Ubuntu 12.04.2 LTS, however you can do similar things on any other Linux distribution.

<!-- more -->

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

# Outbound HTTP
-A OUTPUT -o eth0 -p tcp -m tcp --dport 80 -m state --state NEW -j ACCEPT
-A OUTPUT -o eth0 -p tcp -m tcp --dport 443 -m state --state NEW -j ACCEPT

COMMIT
```

Apply the ruleset with a timeout through `iptables-apply`, and if you lose the connection, fix your rules and try again before continuing.

``` plain 
$ sudo iptables-apply /etc/iptables/rules
Applying new ruleset... done.
Can you establish NEW connections to the machine? (y/N) y
... then my job is done. See you next time.
```

Create the file `/etc/network/if-pre-up.d/iptables`, with the following content. This will automatically load your IPTables rules when you start the server.

``` plain /etc/network/if-pre-up.d/iptables
#!/bin/sh
iptables-restore < /etc/iptables/rules
```

Now give it execute permissions, and execute the file to ensure it loads properly.

``` plain 
$ sudo chmod +x /etc/network/if-pre-up.d/iptables
$ sudo /etc/network/if-pre-up.d/iptables
```

## Fail2ban those wannabe-hackers

Fail2ban is one of my favourite tools when it comes to security, as it will monitor your logfiles, and temporarily ban users that are misusing your resources, be it brute forcing your SSH connection, or DoSing your webserver.

``` plain Install Fail2ban
$ sudo apt-get install fail2ban
[sudo] password for sjones:
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  gamin libgamin0 python-central python-gamin python-support whois
Suggested packages:
  mailx
The following NEW packages will be installed:
  fail2ban gamin libgamin0 python-central python-gamin python-support whois
0 upgraded, 7 newly installed, 0 to remove and 2 not upgraded.
Need to get 254 kB of archives.
After this operation, 1,381 kB of additional disk space will be used.
Do you want to continue [Y/n]? y
...
```

Fail2ban installs a default configuration (`/etc/fail2ban/jail.conf`), but we'll want to make our changes in `/etc/fail2ban/jail.local`, so copy it there.

``` plain 
sudo cp /etc/fail2ban/jail.{conf,local}
```

### Configuration

Change the `ignoreip` line to your IP, and decide on an amount of time to ban the scumbags for (default is 10 minutes). You'll also want to set up a `destemail`, which I normally enter as my own email address followed by `,fail2ban@blocklist.de`. [BlockList.de](http://www.blocklist.de/) is a system to track and automatically report hacking attempts to the proper abuse contact for their IP.

``` plain /etc/fail2ban/jail.local
[DEFAULT]

# "ignoreip" can be an IP address, a CIDR mask or a DNS host
ignoreip = 127.0.0.1/8
bantime  = 600
maxretry = 3

# "backend" specifies the backend used to get files modification. Available
# options are "gamin", "polling" and "auto".
# yoh: For some reason Debian shipped python-gamin didn't work as expected
#      This issue left ToDo, so polling is default backend for now
backend = auto

#
# Destination email address used solely for the interpolations in
# jail.{conf,local} configuration files.
destemail = root@localhost,fail2ban@blocklist.de
```

There are a few other settings you'll want to check out, although the defaults are quite sufficient, so skim through the file quickly until you reach the Actions section.

### Actions

Actions allow you to react to malicious activity, however the default is to issue an IPTables ban, while we want it to ban *and* send an email. Thankfully there is a preconfigured `action_wml`, which does just that.

``` plain /etc/fail2ban/jail.local
# Choose default action.  To change, just override value of 'action' with the
# interpolation to the chosen action shortcut (e.g.  action_mw, action_mwl, etc) in jail.local
# globally (section [DEFAULT]) or per specific section
action = %(action_mwl)s
```

### Jails

In order for Fail2ban to work, it needs to know what to watch. These are configured in the Jails section of the config, and there are quite a few examples pre-loaded and disabled. Since you've only enabled SSH access on the server so far, we'll only enable the SSH and SSH-DDoS jails, however you'll want to add a new jail for each publicly-accessible service that you install on this server.

``` plain /etc/fail2ban/jail.local
[ssh]

enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 6

[ssh-ddos]

enabled  = true
port     = ssh
filter   = sshd-ddos
logpath  = /var/log/auth.log
maxretry = 6
```

### Apply the changes

Now that we've configured Fail2ban, you'll want to reload it, and confirm that it is adding the appropriate rules to IPTables.

``` plain 
$ sudo service fail2ban restart
 * Restarting authentication failure monitor fail2ban
   ...done.

$ sudo iptables -L
Chain INPUT (policy DROP)
target     prot opt source               destination
fail2ban-ssh-ddos  tcp  --  anywhere             anywhere             multiport dports ssh
fail2ban-ssh  tcp  --  anywhere             anywhere             multiport dports ssh
...
Chain fail2ban-ssh (1 references)
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere

Chain fail2ban-ssh-ddos (1 references)
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere
```

At any time, you can use `sudo iptables -L` to list your rules, and subsequently list any currently-banned IPs. At the moment, Fail2ban is handling two malicious individuals:

``` plain Banned IPs
DROP       all  --  204.50.33.22         anywhere
DROP       all  --  195.128.126.114      anywhere
```

## Staying on top of things

You should now have a server that is locked down and ready to use, however this is not the end of your security journey. Stay on top of updates (and always test them in a non-production environment first), always close ports that you don't need, check your logs regularly, and know your servers inside-and-out.

## HackerNews Followup

There are some great comments over at [HackerNews](https://news.ycombinator.com/item?id=6384603), and I suggest you read through them if you're interested in different perspectives and higher security. This post is intended as a beginners guide to security, and the end of this post does not mean your server is invulnerable. Use this to quickly lock down a new server, and then build upon it for your unique situation. You may want to look into IPV6 security, changing your SSH port (security through obscurity), secured kernels (SELinux and GRSecurity), tracking system changes, and full audits if your server has ever been unsecured, or has been online for any substantial length of time. There are hundreds of entry points into a server, and every application you install brings yet another possible hole, but with the proper tools, you can go to bed without worries.
