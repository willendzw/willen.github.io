---
layout: post
title:  "opensips install"
author: Willen
categories: [ SIP ]
tags: [ opensips ]
comments: false
rating: false
---

# **Opensips install**

sudo yum install mysql-server mysql mysql-devel



- yum install https://yum.opensips.org/3.1/releases/el/8/x86_64/opensips-yum-releases-3.1-5.el8.noarch.rpm

- Install and start the OpenSIPS server.

- - Now, you can install OpenSIPS with single command:
    **yum install opensips opensips-cli**
  - Start the OpenSIPS server:
    **systemctl start opensips** - for systemd distributives
    **service opensips start** - for old distributives
  - If you want to start the OpenSIPS server every time on boot, run this:
    **systemctl enable opensips** - for systemd distributives
    **chkconfig opensips on** - for old distributives

https://github.com/OpenSIPS/opensips-cli.git



------------------------------------

Config DB

## Installation

```
$ su - root
$ dnf install mariadb mariadb-server
```

## Initial setup

First let's start MariaDB

```
$ systemctl start mariadb
```

Now start the secure installation assistant

```
$ mysql_secure_installation
```

Press enter if you didn't have setup a password previously

Then it is advisable to answer as follows

```
Set root password? [Y/n] Y
```

[![Warning.png](https://fedoraproject.org/w/uploads/thumb/c/cb/Warning.png/35px-Warning.png)](https://fedoraproject.org/wiki/File:Warning.png)

**Do not use system's root account password**
Do not provide the system administrator's password for your Linux system here. Use a different strong password, since this is a separate authentication for a MySQL user called "root."

```
Set root password? [Y/n] y
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```

To start MariaDB on boot

```
$ systemctl enable mariadb
```

## GUI frontends

There are some popular frontends such as [phpMyAdmin](https://fedoraproject.org/wiki/PhpMyAdmin)

## Default installation and configuration files

The configuration files are stored in the `/etc/my.cnf.d/` directory and the main configuration file is `/etc/my.cnf`

The default log file is `/var/log/mariadb/mariadb.log`

The default installation directory is `/var/lib/mysql`

The default PID file is `/var/run/mariadb/mariadb.pid`

The default unix socket file is `/var/lib/mysql/mysql.sock`

## Firewall

MariaDB operates on port 3306 (or whatever else you set in your `my.cnf`). In firewalld you can open it like this:

```
$ # make it last after reboot
$ firewall-cmd --permanent --add-port=3306/tcp
$ # change runtime configuration
$ firewall-cmd --add-port=3306/tcp
```

In case of iptables:

```
$ iptables -A INPUT -p tcp --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPT
```



