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

### Configure MariaDB

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

Secure mariadb and set root password
```
# mysql_secure_installation
Set root password? [Y/n] Y
New password:                            <MARIADB_ROOT_PASSWORD>
Re-enter new password:                   <MARIADB_ROOT_PASSWORD>
Password updated successfully!
```

Create required librenms DataBase with librenms User having required DB permissions 
```
# mysql -u root -p

CREATE DATABASE librenms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'librenms'@'localhost' IDENTIFIED BY '<MARIADB_USER_PASSWORD>';
GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';
FLUSH PRIVILEGES;
exit

```

Configure PHP-FPM

```
# cp /etc/php-fpm.d/www.conf /etc/php-fpm.d/librenms.conf
# vim /etc/php-fpm.d/librenms.conf
```
Locate and Change [www] to [librenms]

Change user and group to librenms:

```
user = librenms
group = librenms
```
Change listen to a unique name

```
listen = /run/php-fpm-librenms.sock
```

### Configure Apache Web Server

Install mod_ssl apache module
```
# dnf -y install mod_ssl
```

Create SSL certificate
```
# openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout librenms.domain.com.key -out librenms.domain.com.crt
Generating a RSA private key
....................................................................+++++
.....................................................+++++
writing new private key to 'librenms.hitachi-dps.com.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:LK
State or Province Name (full name) []:Western
Locality Name (eg, city) [Default City]:Colombo
Organization Name (eg, company) [Default Company Ltd]: Company Name
Organizational Unit Name (eg, section) []: IT
Common Name (eg, your name or your server's hostname) []: librenms.domain.com
Email Address []:admin@hdomain.com
```

Move generated certificates
```
# mv librenms.domain.com.crt tls/certs/
# mv librenms.domain.com.key tls/private/
```

Configure virtual host in Apache

```
# vim /etc/httpd/conf.d/librenms.conf
<VirtualHost *:80>
  	DocumentRoot /opt/librenms/html/
  	ServerName www.librenms.domain.com
  	ServerAlias  librenms.domain.com
        ErrorLog /var/log/librenms.domain.com_http_error.log
        CustomLog /var/log/librenms.domain.com_http_access.log combined

  AllowEncodedSlashes NoDecode
  <Directory "/opt/librenms/html/">
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
  </Directory>

  # Enable http authorization headers
  <IfModule setenvif_module>
    SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
  </IfModule>

  <FilesMatch ".+\.php$">
    SetHandler "proxy:unix:/run/php-fpm-librenms.sock|fcgi://localhost"
  </FilesMatch>
</VirtualHost>


<VirtualHost *:443>
  	DocumentRoot /opt/librenms/html/
  	ServerName www.librenms.domain.com
  	ServerAlias  librenms.domain.com
        ErrorLog /var/log/librenms.domain.com_https_error.log
        CustomLog /var/log/librenms.domain.com_https_access.log combined

	SSLEngine on
	SSLProtocol all -SSLv2 -SSLv3
	SSLCipherSuite HIGH:!MEDIUM:!aNULL:!MD5:!RC4
	SSLCertificateFile /etc/pki/tls/certs/librenms.domain.com.crt
	SSLCertificateKeyFile /etc/pki/tls/private/librenms.domain.com.key

  AllowEncodedSlashes NoDecode
  <Directory "/opt/librenms/html/">
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
  </Directory>

  # Enable http authorization headers
  <IfModule setenvif_module>
    SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
  </IfModule>

  <FilesMatch ".+\.php$">
    SetHandler "proxy:unix:/run/php-fpm-librenms.sock|fcgi://localhost"
  </FilesMatch>

</VirtualHost>
```

Remove Apache Default configuration

```
# cd /etc/httpd/conf.d/
# mv welcome.conf welcome.conf.bak
# mv ssl.conf ssl.conf.bak
```

Enable https

Add Listen 443 below Listen 80
```
# cd /etc/httpd/conf
# vim httpd.conf
Listen 443
```
