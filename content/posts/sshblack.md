---
author: "Karthik M"
title: "SSHBlack - Securing open linux servers"
date: "2020-01-05"
tags:
- linux
- ssh
- iptables
- security
---

[SSHBlack](http://www.pettingers.org/code/sshblack.html) is a must have utility for open servers in the public
network(internet). Download the latest versions from the website. The installation steps are as follows:

```
$ tar -zxf sshblackv281.tar.gz
$ mv sshblack /usr/local/sshblack
$ cd /usr/local/sshblack
# provide excutable permission for the script
$ chmod 755 sshblack.pl
# Create a new chain BLACKLIST
$ iptables -N BLACKLIST
```

Next we need to update the `sshblack.pl`. Open your favourite editor and update the variables

```
# this will run the process in the background
my($DAEMONIZE) = '1';
# The INPUT log file you want to monitor; If Ubuntu OS its '/var/log/auth.log'; if its RedHat based OS '/var/log/secure';
my($LOG) = '/var/log/auth.log';
# Update your static IP which should never be blacklisted, displayed as WWW.XXX.YYY.ZZZ;
my($LOCALNET) = '^(?:127\.0\.0\.1|WWW\.XXX\.YYY\.ZZZ)';
```

Save the file. `./sshblack.pl` will start the script as a background process. The `/var/log/sshblacklisting` file will
log the IP information of clients accessing/attacking the server.

Once the server is attacked more than 5 times(default value of variable $MAXHITS), a block rule is added to iptables
with the IP information. This prevents new connections to the server from the attacker, in turn preventing the server
from brute force attempts. A sample of IP’s which is blacklisted in my server using the script is listed below:

```
$ iptables -L
Chain BLACKLIST (1 references)
target     prot opt source destination
DROP       all  --  132.232.54.102       anywhere
DROP       all  --  139.59.84.55         anywhere
DROP       all  --  222.187.232.212      anywhere
DROP       all  --  222.187.225.10       anywhere
DROP       all  --  222.187.238.32       anywhere
DROP       all  --  58.241.250.152       anywhere
```

If you clear the iptables, make sure to clear the text database which keeps track of the attacked IP address.
```
$ echo '' > /var/tmp/ssh-blacklist-pending
```
