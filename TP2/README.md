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
