# CentOS7-KnowHow

- [Installation on CentOS](#installation)
  - [Setup Firewall](#setup-firewall)
- [SELinux](#selinux)
- [Systemd](#systemd)
- [Firewall](#firewall)
- [Varia](#varia)


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


# SELinux

## Overview

DAC - Discretionary Access Control (Standard Linux with File System Permissions)
MAC - Manadatory Access Control (SELinux, developed by NSA and Redhat)

- AppArmor is another MAC System
- MAC cannot be bypassed by root
- MAC is based on file attributes (not just file names, e.g. renaming would bypass security)

Modes:
- Enforcing (Rules are enforced, and violations are logged)
- Permissive (violations logged)
- Disabled (SELinux not operational and no logging)

## Read the SELinux mode

getenforce

sestatus

/etc/selinux/config

## Set the SELinux mode

setenforce 0

## Disable SELinux

Permanently disable SE-Linux

## Prevent Runtime Changes to Mode

setsebool secure_mode_policyload on (-P)  # -P to persist
 


```bash
sed -i /etc/selinux/config -r -e 's/^SELINUX=.*/SELINUX=disabled/g'
reboot
```


View the SELinux context of an object
ls -Zd .

## Booleans

getsebool -a  # list the transient booleans from the loaded policy in the memory

semanange boolean --list # list the transient and the persistent settings

semanage boolean -l | grep secure_mode_policyload

semanage secure_mode_policyload on # set the boolean in memory



/etc/selinux/conf

/etc/selinux/targeted/contexts/files

getenforce
setenforce 1
sestatus

ls -Z <file>
ps -Z -p $(pgrep sshd)
id -Z

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

firewall-cmd --get-active-zones
firewall-cmd --list-all-zones
```
Add a service
```firewall-cmd --add-service=http

Restart firewalld on CentOS 7

Add a service permanent
```firewall-cmd --add-service=http --permanent




# Varia

```
hostnamectl
```



