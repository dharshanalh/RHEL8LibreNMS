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
dnf -y install epel-release
```

Reset PHP module in yum repository to default stream.

```
dnf module reset php
```

Enable PHP 7.3

```
dnf module enable php:7.3
```

Install required packeges

```
dnf install bash-completion cronie fping git httpd ImageMagick mariadb-server mtr net-snmp net-snmp-utils nmap php-fpm php-cli php-common php-curl php-gd php-gmp php-json php-mbstring php-process php-snmp php-xml php-zip php-mysqlnd python3 python3-PyMySQL python3-redis python3-memcached python3-pip python3-systemd rrdtool unzip
```
