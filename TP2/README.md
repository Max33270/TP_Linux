# TP2 : Gestion de service

## I. Un premier serveur web

### 1. Installation

```
[max@localhost ~]$ sudo dnf install httpd
``` 

```
[max@localhost ~]$ sudo vim /etc/httpd/conf/httpd.conf
```

On supprime tout les commentaires avec :
``` 
:g/^ *#.*/d
```


```
[max@localhost ~]$ sudo systemctl start httpd
``` 

```
[max@localhost ~]$ sudo systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
``` 

```
[max@localhost ~]$ sudo ss -alnpt
[sudo] password for max:
State  Recv-Q  Send-Q   Local Address:Port   Peer Address:Port Process
LISTEN 0       128            0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=683,fd=3))
LISTEN 0       128               [::]:22             [::]:*     users:(("sshd",pid=683,fd=4))
LISTEN 0       511                  *:80                *:*     users:(("httpd",pid=10766,fd=4),("httpd",pid=10765,fd=4),("httpd",pid=10764,fd=4),("httpd",pid=10762,fd=4))
``` 

```
[max@localhost ~]$ sudo firewall-cmd --zone=public --permanent --add-service=http
success
[max@localhost ~]$ sudo firewall-cmd --zone=public --permanent --add-port 80/tcp
success
[max@localhost ~]$ sudo firewall-cmd --reload
success
[max@localhost ~]$
``` 

```
[max@localhost ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client http ssh
  ports: 22/tcp 80/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
``` 

```
[max@localhost ~]$ sudo systemctl is-active httpd
active
[max@localhost ~]$ sudo systemctl is-enabled httpd
enabled
[max@localhost ~]$ curl 10.102.1.11:80
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
      .................
``` 

```
PS C:\Users\mdoub> curl 10.102.1.11:80
curl : HTTP Server Test Page
This page is used to test the proper operation of an HTTP server after it has been installed on a Rocky Linux system.
If you can read this page, it means that the software it working correctly.
Just visiting?
................
``` 

### 2. Avancer vers la maîtrise du service

```
[max@localhost testpage]$ sudo systemctl status httpd

[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true
OOMPolicy=continue

[Install]
WantedBy=multi-user.target
``` 


```
[max@localhost ~]$ sudo cat /etc/httpd/conf/httpd.conf

Listen 80

Include conf.modules.d/*.conf

User apache

```

```
[max@localhost ~]$ ps -ef

root       10762       1  0 10:11 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     10763   10762  0 10:11 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     10764   10762  0 10:11 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     10765   10762  0 10:11 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     10766   10762  0 10:11 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
``` 

```
[max@localhost ~]$ cd /usr/share/testpage/
[max@localhost testpage]$ ls -al
total 12
drwxr-xr-x.  2 root root   24 Nov 15 10:01 .
drwxr-xr-x. 82 root root 4096 Nov 15 10:01 ..
-rw-r--r--.  1 root root 7620 Jul  6 04:37 index.html
``` 

```
[max@localhost testpage]$ sudo useradd mdoub -m -d /usr/share/httpd -s /sbin/nologin
useradd: warning: the home directory /usr/share/httpd already exists.
useradd: Not copying any file from skel directory into it.
``` 

```
[max@localhost testpage]$ cat /etc/passwd

apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
mdoub:x:1001:1001::/usr/share/httpd:/sbin/nologin
``` 

```
[max@localhost testpage]$ sudo passwd mdoub
Changing password for user mdoub.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
``` 

```
[max@localhost testpage]$ sudo cat /etc/httpd/conf/httpd.conf

ServerRoot "/etc/httpd"

Listen 80

Include conf.modules.d/*.conf

User mdoub
Group mdoub
```

```
[max@localhost testpage]$ sudo systemctl restart httpd
``` 

```
[max@localhost testpage]$ ps -ef

root       11160       1  0 11:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
mdoub      11161   11160  0 11:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
mdoub      11162   11160  0 11:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
mdoub      11163   11160  0 11:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
mdoub      11164   11160  0 11:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
``` 

```
[max@localhost testpage]$ sudo cat /etc/httpd/conf/httpd.conf

ServerRoot "/etc/httpd"

Listen 33
``` 

```
[max@localhost testpage]$ sudo firewall-cmd --zone=public --permanent --add-port 33/tcp
success
[max@localhost testpage]$ sudo firewall-cmd --zone=public --permanent --remove-port 80/tcp
success
[max@localhost testpage]$ sudo firewall-cmd --reload
success
[max@localhost testpage]$ sudo firewall-cmd --zone=public --permanent --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: cockpit dhcpv6-client http ssh
  ports: 22/tcp 33/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
``` 

```
[max@localhost testpage]$ sudo systemctl restart httpd
``` 

```
[max@localhost testpage]$ ss -alpnt
State                  Recv-Q                 Send-Q                                 Local Address:Port                                 Peer Address:Port                Process
LISTEN                 0                      511                                                *:33                                              *:*
```

```
[max@localhost testpage]$ curl 10.102.1.11:33
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/
      .........
```

```
PS C:\Users\mdoub> curl 10.102.1.11:33
curl : HTTP Server Test Page
This page is used to test the proper operation of an HTTP server after it has been installed on a Rocky Linux system.
If you can read this page, it means that the software it working correctly.
Just visiting?
``` 

## II. Une stack web plus avancée

### 1. Intro blabla

Configuration Apache Reset

### 2. Setup

#### A. Base de données

```
[max@db ~]$ sudo dnf install mariadb-server

Complete!
``` 

```
[max@db ~]$ sudo systemctl enable mariadb
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
[max@db ~]$ sudo systemctl start mariadb
[max@db ~]$ mysql_secure_installation

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

``` 

```
[max@db ~]$ sudo ss -alpnt
[sudo] password for max:
State  Recv-Q  Send-Q   Local Address:Port   Peer Address:Port Process
LISTEN 0       80                   *:3306              *:*     users:(("mariadbd",pid=12798,fd=19))
``` 

```
[max@db ~]$ sudo firewall-cmd --zone=public --permanent --add-port 3306/tcp
success
``` 

```
[max@db ~]$ sudo mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 15
Server version: 10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE USER 'nextcloud'@'10.102.1.11' IDENTIFIED BY 'pewpewpew';
Query OK, 0 rows affected (0.004 sec)

MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf4_general_ci;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.102.1.11';
Query OK, 0 rows affected (0.004 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)
``` 

```
[max@localhost ~]$ mysql -u nextcloud -h 10.102.1.12 -p
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 23
Server version: 10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| nextcloud          |
+--------------------+
2 rows in set (0.001 sec)

MariaDB [(none)]> USE nextcloud
Database changed
MariaDB [nextcloud]> SHOW TABLES;
Empty set (0.001 sec)   
```

```
MariaDB [(none)]> SELECT user FROM mysql.user
``` 

#### B. Serveur Web et NextCloud

```
[max@localhost ~]$ sudo dnf config-manager --set-enabled crb

[max@localhost ~]$ sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
Installed:
  epel-release-9-4.el9.noarch                remi-release-9.0-6.el9.remi.noarch
  yum-utils-4.0.24-4.el9_0.noarch

Complete!

[max@localhost ~]$ dnf module list php
Remi's Modular repository for Enterprise Linux 9 - x86_64
Name        Stream          Profiles                          Summary
php         remi-7.4        common [d], devel, minimal        PHP scripting language
php         remi-8.0        common [d], devel, minimal        PHP scripting language
php         remi-8.1        common [d], devel, minimal        PHP scripting language
php         remi-8.2        common [d], devel, minimal        PHP scripting language

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled

[max@localhost ~]$ sudo dnf module enable php:remi-8.1 -y
Enabling module streams:
 php                                      remi-8.1

Transaction Summary
===========================================================================================

Complete!

[max@localhost ~]$ sudo dnf install -y php81-php
Installed:
  environment-modules-5.0.1-1.el9.x86_64       libsodium-1.0.18-8.el9.x86_64
  libxslt-1.1.34-9.el9.x86_64                  oniguruma5php-6.9.8-1.el9.remi.x86_64
  php81-php-8.1.12-1.el9.remi.x86_64           php81-php-cli-8.1.12-1.el9.remi.x86_64
  php81-php-common-8.1.12-1.el9.remi.x86_64    php81-php-fpm-8.1.12-1.el9.remi.x86_64
  php81-php-mbstring-8.1.12-1.el9.remi.x86_64  php81-php-opcache-8.1.12-1.el9.remi.x86_64
  php81-php-pdo-8.1.12-1.el9.remi.x86_64       php81-php-sodium-8.1.12-1.el9.remi.x86_64
  php81-php-xml-8.1.12-1.el9.remi.x86_64       php81-runtime-8.1-2.el9.remi.x86_64
  scl-utils-1:2.0.3-2.el9.x86_64               tcl-1:8.6.10-6.el9.x86_64

Complete!
``` 

```
[max@localhost ~]$ sudo dnf install -y libxml2 openssl php81-php php81-php-ctype php81-php-curl php81-php-gd php81-php-iconv php81-php-json php81-php-libxml php81-php-mbstring php81-php-openssl php81-php-posix php81-php-session php81-php-xml php81-php-zip php81-php-zlib php81-php-pdo php81-php-mysqlnd php81-php-intl php81-php-bcmath php81-php-gmp
Installed:
  fontconfig-2.13.94-2.el9.x86_64              fribidi-1.0.10-6.el9.x86_64
  gd3php-2.3.3-5.el9.remi.x86_64               jbigkit-libs-2.1-23.el9.x86_64
  libX11-1.7.0-7.el9.x86_64                    libX11-common-1.7.0-7.el9.noarch
  libXau-1.0.9-8.el9.x86_64                    libXpm-3.5.13-7.el9.x86_64
  libicu71-71.1-2.el9.remi.x86_64              libimagequant-2.17.0-1.el9.x86_64
  libjpeg-turbo-2.0.90-5.el9.x86_64            libraqm-0.8.0-1.el9.x86_64
  libtiff-4.2.0-3.el9.x86_64                   libwebp-1.2.0-3.el9.x86_64
  libxcb-1.13.1-9.el9.x86_64                   php81-php-bcmath-8.1.12-1.el9.remi.x86_64
  php81-php-gd-8.1.12-1.el9.remi.x86_64        php81-php-gmp-8.1.12-1.el9.remi.x86_64
  php81-php-intl-8.1.12-1.el9.remi.x86_64      php81-php-mysqlnd-8.1.12-1.el9.remi.x86_64
  php81-php-pecl-zip-1.21.1-1.el9.remi.x86_64  php81-php-process-8.1.12-1.el9.remi.x86_64
  remi-libzip-1.9.2-3.el9.remi.x86_64          xml-common-0.6.3-58.el9.noarch

Complete!
``` 

```
[max@localhost ~]$ sudo mkdir /var/www/tp2_nextcloud
``` 

```
[max@localhost ~]$ sudo dnf install wget -y
Installed:
  wget-1.21.1-7.el9.x86_64

Complete!

[max@localhost ~]$ wget https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip

nextcloud-25.0.0rc3.zi 100%[===========================>] 168.17M  2.89MB/s    in 89s

2022-11-15 14:49:02 (1.89 MB/s) - ‘nextcloud-25.0.0rc3.zip’ saved [176341139/176341139]
```

```
[max@localhost ~]$ sudo dnf install unzip -y
Installed:
  unzip-6.0-56.el9.x86_64

Complete!

[max@localhost ~]$ sudo unzip nextcloud-25.0.0rc3.zip
   creating: nextcloud/config/
 extracting: nextcloud/config/CAN_INSTALL
  inflating: nextcloud/config/config.sample.php
  inflating: nextcloud/config/.htaccess

[max@localhost ~]$ sudo mv nextcloud/* /var/www/tp2_nextcloud/
```

```
[max@localhost ~]$ ls /var/www/tp2_nextcloud/
3rdparty  console.php  dist        occ           public.php  status.php
apps      COPYING      index.html
``` 

```
[max@localhost ~]$ ls -al /var/www/tp2_nextcloud/
total 132
drwxr-xr-x. 14 apache root  4096 Nov 15 14:53 .
drwxr-xr-x.  5 root   root    54 Nov 15 14:45 ..
drwxr-xr-x. 47 apache root  4096 Oct  6 14:47 3rdparty
drwxr-xr-x. 50 apache root  4096 Oct  6 14:44 apps
-rw-r--r--.  1 apache root 19327 Oct  6 14:42 AUTHORS
drwxr-xr-x.  2 apache root    67 Oct  6 14:47 config
``` 

```
[max@localhost ~]$ sudo cat /etc/httpd/conf/httpd.conf

IncludeOptional conf.d/*.conf
```

```
[max@localhost ~]$ sudo systemctl restart httpd
```







