# RHEL8 LibreNMS installation

## Install RHEL 8 in LibreNMS

Become super user

```
# sudo -s
```

Update yum cache

```
# dnf update -y
```

Check current Linux kernal

```
# uname -r
5.4.17-2102.201.3.el8uek.x86_64
```

Check Redhat release

```
# cat /etc/redhat-release
Red Hat Enterprise Linux release 8.5 (Ootpa)
```

Install EPEL (Extra Packages for Enterprise Linux)

```
# dnf -y install epel-release
```
Reset PHP module in yum repository to default stream.

```
# dnf module reset php
```

Enable PHP 7.3

```
# dnf module enable php:7.3
```

Install Required Packages

```
# dnf install bash-completion cronie fping git httpd ImageMagick mariadb-server mtr net-snmp net-snmp-utils nmap php-fpm php-cli php-common php-curl php-gd php-gmp php-json php-mbstring php-process php-snmp php-xml php-zip php-mysqlnd python3 python3-PyMySQL python3-redis python3-memcached python3-pip python3-systemd rrdtool unzip
```

Add librenms user

```
# useradd librenms -d /opt/librenms -M -r -s "$(which bash)"
# cat /etc/passwd | grep librenms
librenms:x:988:984::/opt/librenms:/bin/bash
```

Download LibreNMS
```
# cd /opt
# git clone https://github.com/librenms/librenms.git
```

Set permissions
```
# chown -R librenms:librenms /opt/librenms
# ls -als
4 drwxr-xr-x. 24 librenms librenms 4096 Apr  7 13:48 librenms

# chmod 771 /opt/librenms
# ls -als
4 drwxrwx--x. 24 librenms librenms 4096 Apr  7 13:48 librenms

# setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
# setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/

# getfacl /opt/librenms/rrd
getfacl: Removing leading '/' from absolute path names
file: opt/librenms/rrd
owner: librenms
group: librenms
user::rwx
group::rwx
other::r-x
default:user::rwx
default:group::rwx
default:other::r-x

```

Install PHP dependencies

```
# su - librenms
# ./scripts/composer_wrapper.php install --no-dev
# exit
```

Set timezone

Find and set following variable

```
# vim /etc/php.ini
date.timezone = Asia/Colombo
```

Set the same timezone in Linux operating system

```
# timedatectl set-timezone Asia/Colombo
# timedatectl show
Timezone=Asia/Colombo
LocalRTC=no
CanNTP=yes
NTP=no
NTPSynchronized=no
TimeUSec=Thu 2022-04-07 14:05:10 +0530
RTCTimeUSec=Thu 2022-04-07 14:05:10 +0530
```

Configure MariaDB

Within the [mysqld] section add following two parameters

```
# vim /etc/my.cnf.d/mariadb-server.cnf
innodb_file_per_table=1
lower_case_table_names=0
```

```
# systemctl enable mariadb
# systemctl restart mariadb

OR

# systemctl enable --now mariadb.service
```
