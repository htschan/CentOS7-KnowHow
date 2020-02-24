# CentOS7-KnowHow

- [Installation on CentOS](#installation)
  - [Setup Firewall](#setup-firewall)
- [yum](#yum)
- [SELinux](#selinux)
- [Systemd](#systemd)
- [Firewall](#firewall)
- [Varia](#varia)
- [vi](#vi)
- [Grep](#grep)
- [sed](#sed)
- [awk](#awk)
- [join](#join)



# Installation on CentOS

## Setup Users and SSH

Use PuTTYgen to generate SSG-Keypairs.
Paste the public key into the file 'authorized_keys'. The key must start with "ssh-rsa AAAA ...."

```
$ su - <the User>
$ mkdir .ssh
$ chmod 700 .ssh

$ vi .ssh/authorized_keys
$ chmod 600 .ssh/authorized_keys
$ exit
```
Once SSH login is setup, disable Username/Password login:

sudo vim /etc/ssh/sshd_config

```
[...]
PasswordAuthentication no
[...]
UsePAM no
[...]
```
`systemctl reload sshd`


https://www.digitalocean.com/community/tutorials/initial-server-setup-with-centos-7

## Set Timezone and setup NTP

`sudo timedatectl set-timezone region/timezone`

```
sudo yum install ntp
sudo systemctl start ntpd
sudo systemctl enable ntpd
```

## Install Midnight Commander

`sudo yum -y install mc`

### Install Docker

https://docs.docker.com/engine/installation/linux/docker-ce/centos/#upgrade-docker-ce-1


## Install mariadb
https://support.rackspace.com/how-to/installing-mysql-server-on-centos/


`sudo yum install mariadb-server mariadb`

## Install git

https://www.digitalocean.com/community/tutorials/how-to-install-git-on-centos-7

## Install NodeJS

https://nodejs.org/en/download/package-manager/

## Setup Firewall

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7

## Install Nginx

https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7

Default Homepage
`/usr/share/nginx/html`

SE-Linux Status
`sestatus`

Persist httpd-SELinux Flag
`setsebool httpd_can_network_connect on -P`

List available SELinux booleans
`getsebool -a | grep httpd` 


## Setup SSL and Certificate with Nginx

https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-7


Create crontab job to renew the certificate

`30 2 * * 1 /usr/bin/certbot renew >> /var/log/le-renew.log`
`35 2 * * 1 /usr/bin/systemctl reload nginx`

Renew manually
sudo certbot renew certonly -a webroot --webroot-path=/usr/share/nginx/html -d timepuncher.ch

Refer to nginx.conf sample.

### Test your SSL server

https://www.ssllabs.com/ssltest/analyze.html?d=timepuncher.ch

### How to setup an NodeJS application

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-centos-7

# Yum

Download a package without installing it:

```
yum install --downloadonly --downloaddir=<directory> <package>
```

## yumdownloader

If downloading a installed package, "yumdownloader" is useful.

Install the yum-utils package:

```
yum install yum-utils
```

Run the command followed by the desired package:

```
yumdownloader <package>
```
- The package is saved in the current working directly by default; use the --destdir option to specify an alternate location.
- Be sure to add --resolve if you need to download dependencies.

Refer also to [How to use yum to download a package without installing it](https://access.redhat.com/solutions/10154)

# SELinux

## Overview

DAC - Discretionary Access Control (Standard Linux with File System Permissions)

MAC - Mandatory Access Control (SELinux, developed by NSA and Redhat)

- AppArmor is another MAC System
- MAC cannot be bypassed by root
- MAC is based on file attributes (not just file names, e.g. renaming would bypass security)

Modes:
- Enforcing (Rules are enforced, and violations are logged)
- Permissive (violations logged)
- Disabled (SELinux not operational and no logging)

## References

[SELinux User's Guide and Administration (RHEL 7)](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/selinux_users_and_administrators_guide/index)

## Tools

These tools are installed by default:

- semanage
- sestatus

These tools are required to customize and diagnose SELinux

- policycoreutils
- policycoreutils-devel
- setools-console
- attr

To install the tools execute this command:

```bash
yum install policycoreutils policycoreutils-devel setools-console.x86_64 attr
```

attr gives access to getfattr

## Read the SELinux mode

getenforce

sestatus

/etc/selinux/config

## Set the SELinux mode

setenforce 0

## Disable SELinux

Permanently disable SE-Linux

```bash
sed -i /etc/selinux/config -r -e 's/^SELINUX=.*/SELINUX=disabled/g'
reboot
```

## Prevent Runtime Changes to Mode

setsebool secure_mode_policyload on (-P)  # -P to persist
 
## Context (Label)

View the SELinux context of an object

```
ls -Zd .
```

## Booleans

```
getsebool -a  # list the transient booleans from the loaded policy in the memory
```

semanange boolean --list # list the transient and the persistent settings

semanage boolean -l | grep secure_mode_policyload

semanage secure_mode_policyload on # set the boolean in memory

## Type Enforcement

Display the SELinux context of a process:

```
ps -Zp $(pgrep httpd)
```

```
➜  pj ps -Zp $(pgrep httpd)
LABEL                              PID TTY      STAT   TIME COMMAND
system_u:system_r:httpd_t:s0     75164 ?        Ss     0:01 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0     75165 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0     75166 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0     75167 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0     75168 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0     75169 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND

```

For the Targeted Mode, only the Type, e.g. `httpd_t` is of interest.
The user, role and MLS-range field are not used in Targeted Mode.

Display the SELinux context of a directory:

```
ls -dZ /var/www/html
```

This can alos be achieved using `getfattr`:

```
getfattr -n security.selinux /var/www/html
```
## Search for denials

```
ausearch -m avc -c httpd

aureport -a
```


## Allowing access

```
audit2allow -a -M mycertwatch

#******************** IMPORTANT ***********************
#To make this policy package active, execute:

semodule -i mycertwatch.pp
```

Produce human readable description:
```
audit2allow -w -a
```

SE Alert Message

Alerts are logged in /var/log/messages. To view a description, execute:

```
sealert -l 84e0b04d-d0ad-4347-8317-22e74f6cd020
````

## Configuration files

/etc/selinux/conf

/etc/selinux/targeted/contexts/files

getenforce

setenforce 1

sestatus

```
ls -Z <file>
ps -Z -p $(pgrep sshd)
id -Z
```

aussearch -m avc -ts recent

restorecon /etc/shadow

chcon -t shadow_t /etc/shadow

getsebool -a

semanage boolean -l

setsebool sdfsf on (temporary setting)

setsebool -P sdfsdf on (permanent setting)

semanage port -l

semanage port -a -t ssh_port_t  -p tcp 2022

# Systemd


# Firewall

```
firewall-cmd --list-all
```

```
firewall-cmd --get-active-zones
firewall-cmd --list-all-zones
```

Add a service

```
firewall-cmd --add-service=http
firewall-cmd --runtime-to-permanent
```

Restart firewalld on CentOS 7

Add a service permanent

```
firewall-cmd --add-service=http --permanent
```



# Varia

```
hostnamectl
```


# Bash

Esc + .   or Alt + .
Get last argument

Ctrl + r to search for command

Go back to last directory

```
cd -
```

## Job control

While editing in vi, press Ctrl + z to send vi to the background.

To get a list of jobs:

```
jobs
```

To resume a specific background job:

```
fg 3
```


## Files and directories

List files only in directory

```
ls <directory> | grep -v '[/@=|]$' 
```

## Run command again as sudo

Just execute last command as sudo:

```
sudo !!
```

## Calculator bc

[bc Excamples](https://www.geeksforgeeks.org/bc-command-linux-examples/)

Set number of decimal digits example:

```
echo "scale=4;10/3" | bc -l
```


# vi

## Navigation

- Moving Cursor: up / down / left / right -> k / j / h / l
- Moving between words: Move right one word / Move left one word: w / b
- Moving pagewise: Move forward / Move backward: Ctrl + f / Ctrl + b
- Moving halfpagewise: Move forward / Move backward: Ctrl + d / Ctrl + u
- Start of file: `[[` or :1
- End of file: `]]` or SHIFT + g
- Page Up / Page Down -> CTRL + F / CTRL + B
- Move to start of line: 0 (zero)
- Move to end of line: $
- Search for text forward: `/searchstring`
- Remove highlighting: :nohl
- Find and Replace: :s/s1/s2

## Editing

### Inserting

- a - append current position
- A - append end of line
- i - insert current position
- I - insert beginning of line
- R - reeplace current position
- o - start new line, below current position
- O - start new line, above current position

### Copy and Pasting

- yy -copy current line to a buffer
- p - paster line in buffer below current line
- n p - n is a number, past n times

Multiple lines

- v to start selecting
- y to copy
- d to cut
- p to paste

### Undo

- u - undo last edit

### Deleting Lines

- dd - delete on line
- n d - delete n lines of text

### Deleting Characters

- x - delete character to the right
- X - delete character to the left

## Edit multiple Files

vi -O file1 file2

Switch between files: 
- right window: Ctrl + w l 
- left window: Ctlr + w h


# Grep

Search for text with regular Expressions.

Search for lines beginning with `a` while ignoring case:

```
grep -i ^a file.tx
```

Search for lines beginning with `a` or `b` while ignoring case:

```
grep -i ^[ab] file.txt
```

Invert match:

```
grep -v -i ^[ab] file.txt
```

Multiple search patterns, begin with `a` or `b` or end with `inc`:

```
grep -e ^[ab] -e inc$ file.txt
```

Context line control, output `<num>` trailing lines:

```
grep -A <num> <pattern> <file>
```

and output `<num>` lines before match:
  
```
grep -B <num> <pattern> <file>
```

# sed


# awk


# join






