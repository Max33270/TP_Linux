# TP3 : Amélioration de la solution NextCloud - Maxim Doublait B2B

## Module 6 : Automatisation

### Machine 10.102.1.11/24 (web)

```
[max@web ~]$ sudo nano tp3_automatisation.sh

#!/bin/bash

dnf config-manager --set-enabled crb -y
dnf install dnf-utils http://rpms.remirepo.net/entreprise/remi-release-9.rpm -y
dnf module list php -y
dnf module enable php:remi-8.1 -y
dnf install -y php81-php -y
dnf install -y libxml2 openssl php81-php php81-php-ctype php81-php-curl php81-php-gd php81-php-iconv php81-php-json php81-php-libxml php81-php-mbstring php81-php-openssl php81-php-posix php81-php-session php81-php-xml php81-php-zip php81-php-zlib php81-php-pdo php81-php-mysqlnd php81-php-intl php81-php-bcmath php81-php-gmp
mkdir /var/www/tp2_nextcloud
dnf install wget -y
wget https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip
dnf install unzip -y
unzip nextcloud-25.0.0rc3.zip
mv nextcloud/* /var/www/tp2_nextcloud
systemctl restart httpd
```

```
[max@web ~]$ sudo chmod +x tp3_automatisation.sh
[max@web ~]$ sudo ./tp3_automatisation.sh

Rocky Linux 9 - BaseOS                                   7.3 kB/s | 3.6 kB     00:00
Rocky Linux 9 - AppStream                                 10 kB/s | 3.6 kB     00:00
Rocky Linux 9 - CRB                                       12 kB/s | 3.6 kB     00:00
[MIRROR] remi-release-9.rpm: Status code: 404 for http://rpms.remirepo.net/entreprise/remi-release-9.rpm (IP: 109.238.14.107)
[MIRROR] remi-release-9.rpm: Status code: 404 for http://rpms.remirepo.net/entreprise/remi-release-9.rpm (IP: 109.238.14.107)
[MIRROR] remi-release-9.rpm: Status code: 404 for http://rpms.remirepo.net/entreprise/remi-release-9.rpm (IP: 109.238.14.107)
[MIRROR] remi-release-9.rpm: Status code: 404 for http://rpms.remirepo.net/entreprise/remi-release-9.rpm (IP: 109.238.14.107)
[FAILED] remi-release-9.rpm: Status code: 404 for http://rpms.remirepo.net/entreprise/remi-release-9.rpm (IP: 109.238.14.107)
Status code: 404 for http://rpms.remirepo.net/entreprise/remi-release-9.rpm (IP: 109.238.14.107)
Last metadata expiration check: 0:00:02 ago on Mon 21 Nov 2022 12:56:57 AM CET.
Remi's Modular repository for Enterprise Linux 9 - x86_64
Name       Stream             Profiles                        Summary
php        remi-7.4           common [d], devel, minimal      PHP scripting language
php        remi-8.0           common [d], devel, minimal      PHP scripting language
php        remi-8.1 [e]       common [d], devel, minimal      PHP scripting language
php        remi-8.2           common [d], devel, minimal      PHP scripting language

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
Last metadata expiration check: 0:00:03 ago on Mon 21 Nov 2022 12:56:57 AM CET.
Dependencies resolved.
Nothing to do.
Complete!
Last metadata expiration check: 0:00:04 ago on Mon 21 Nov 2022 12:56:57 AM CET.
Package php81-php-8.1.12-1.el9.remi.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
Last metadata expiration check: 0:00:05 ago on Mon 21 Nov 2022 12:56:57 AM CET.
Package libxml2-2.9.13-1.el9_0.1.x86_64 is already installed.
Package openssl-1:3.0.1-43.el9_0.x86_64 is already installed.
Package php81-php-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-gd-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-mbstring-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-process-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-xml-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-pecl-zip-1.21.1-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-pdo-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-mysqlnd-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-intl-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-bcmath-8.1.12-1.el9.remi.x86_64 is already installed.
Package php81-php-gmp-8.1.12-1.el9.remi.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
mkdir: cannot create directory ‘/var/www/tp2_nextcloud’: File exists
Last metadata expiration check: 0:00:06 ago on Mon 21 Nov 2022 12:56:57 AM CET.
Package wget-1.21.1-7.el9.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
--2022-11-21 00:57:03--  https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip
Resolving download.nextcloud.com (download.nextcloud.com)... 95.217.64.181, 2a01:4f9:2a:3119::181
Connecting to download.nextcloud.com (download.nextcloud.com)|95.217.64.181|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 176341139 (168M) [application/zip]
Saving to: ‘nextcloud-25.0.0rc3.zip.6’

nextcloud-25.0.0rc3.zi 100%[=========================>] 168.17M  2.64MB/s    in 76s

2022-11-21 00:58:20 (2.20 MB/s) - ‘nextcloud-25.0.0rc3.zip.6’ saved [176341139/176341139]

Last metadata expiration check: 0:01:23 ago on Mon 21 Nov 2022 12:56:57 AM CET.
Package unzip-6.0-56.el9.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
Archive:  nextcloud-25.0.0rc3.zip
  inflating: nextcloud/index.php
replace nextcloud/ocs-provider/index.php? [y]es, [n]o, [A]ll, [N]one, [r]ename: N
  inflating: nextcloud/remote.php
  inflating: nextcloud/index.html
 extracting: nextcloud/robots.txt
  inflating: nextcloud/occ
  inflating: nextcloud/AUTHORS
  inflating: nextcloud/version.php
  inflating: nextcloud/cron.php
  inflating: nextcloud/console.php
  inflating: nextcloud/status.php
  inflating: nextcloud/public.php
  inflating: nextcloud/COPYING
mv: cannot move 'nextcloud/3rdparty' to '/var/www/tp2_nextcloud/3rdparty': File exists
mv: cannot move 'nextcloud/apps' to '/var/www/tp2_nextcloud/apps': File exists
mv: cannot move 'nextcloud/config' to '/var/www/tp2_nextcloud/config': File exists
mv: cannot move 'nextcloud/core' to '/var/www/tp2_nextcloud/core': File exists
mv: cannot move 'nextcloud/dist' to '/var/www/tp2_nextcloud/dist': File exists
mv: cannot move 'nextcloud/lib' to '/var/www/tp2_nextcloud/lib': File exists
mv: cannot move 'nextcloud/ocm-provider' to '/var/www/tp2_nextcloud/ocm-provider': File exists
mv: cannot move 'nextcloud/ocs' to '/var/www/tp2_nextcloud/ocs': File exists
mv: cannot move 'nextcloud/ocs-provider' to '/var/www/tp2_nextcloud/ocs-provider': File exists
mv: cannot move 'nextcloud/resources' to '/var/www/tp2_nextcloud/resources': File exists
mv: cannot move 'nextcloud/themes' to '/var/www/tp2_nextcloud/themes': File exists
mv: cannot move 'nextcloud/updater' to '/var/www/tp2_nextcloud/updater': File exists
``` 

### Machine 10.102.1.12/24 (web)
```
[max@db ~]$ sudo nano tp3_automation_nextcloud.sh

#!/bin/bash

dnf install httpd -y
systemctl start httpd
systemctl enable httpd
dnf install mariadb-server -y
systemctl enable mariadb
systemctl start mariadb
dnf install expect -y
firewall-cmd --zone=public --permanent --add-service=http
firewall-cmd --zone=public --permanent --add-port 80/tcp
firewall-cmd --reload
useradd mdoub -m -d /usr/share/httpd -s /sbin/nologin
echo "root" | passwd mdoub --stdin
echo "root" | passwd mdoub --stdin
cat /etc/httpd/conf/newhttpd.conf > /etc/httpd/conf/httpd.conf
systemctl restart httpd
firewall-cmd --zone=public --permanent --add-port 33/tcp
firewall-cmd --zone=public --permanent --remove-port 80/tcp
firewall-cmd --zone=public --permanent --add-port 3306/tcp
firewall-cmd --reload
mysql -uroot < sql_file.sql
``` 

```
[max@db ~]$ sudo nano sql_file.sql

CREATE USER if not exists 'nextcloud'@'10.102.1.11' IDENTIFIED BY 'pewpewpew';
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4;
GRANT ALL PRIVILEGES on nextcloud.* TO 'nextcloud'@'10.102.1.11';
FLUSH PRIVILEGES;
``` 

```
[max@db ~]$ sudo chmod +x tp3_automation_nextcloud.sh
[max@db ~]$ sudo chmod +x sql_file.sql
``` 

```
[max@db ~]$ sudo ./tp3_automation_nextcloud.sh

Last metadata expiration check: 2:15:46 ago on Sun 20 Nov 2022 10:49:24 PM CET.
Package httpd-2.4.51-7.el9_0.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
Last metadata expiration check: 2:15:47 ago on Sun 20 Nov 2022 10:49:24 PM CET.
Package mariadb-server-3:10.5.16-2.el9_0.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
Last metadata expiration check: 2:15:48 ago on Sun 20 Nov 2022 10:49:24 PM CET.
Package expect-5.45.4-15.el9.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
Warning: ALREADY_ENABLED: http
success
success
success
useradd: user 'mdoub' already exists
Changing password for user mdoub.
passwd: all authentication tokens updated successfully.
Changing password for user mdoub.
passwd: all authentication tokens updated successfully.
Warning: ALREADY_ENABLED: 33:tcp
success
success
Warning: ALREADY_ENABLED: 3306:tcp
success
success
``` 

## Ajouter dans le fichier hosts de votre pc la ligne : 10.102.1.11 web.tp2.linux 
```
#
# Copyright (c) 2007 F-Secure Corporation 
# 
# This is a HOSTS file created during malware removal. 
#
# Your original HOSTS file was infected and it was replaced 
# by this file containing only clean default entries. 
# The original HOSTS file may be restored from the product's
# quarantine feature.
#
127.0.0.1	localhost
10.102.1.11 web.tp2.linux
::1			localhost
```



