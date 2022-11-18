# TP3 : Amélioration de la solution NextCloud - Maxim Doublait B2B

## Module 2 : Réplication de base de données

### Installations

```
[max@replication ~]$ sudo dnf install mariadb-server

Complete!
``` 

```
[max@replication ~]$ sudo systemctl start mariadb
[sudo] password for max:
sudo: a password is required
[max@replication ~]$ systemctl start mariadb
Failed to start mariadb.service: Access denied
See system logs and 'systemctl status mariadb.service' for details.
[max@replication ~]$ sudo systemctl start mariadb
[sudo] password for max:
[max@replication ~]$ sudo systemctl enable mariadb
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
[max@replication ~]$ sudo systemctl status mariadb
● mariadb.service - MariaDB 10.5 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disable>
     Active: active (running) since Fri 2022-11-18 19:25:55 CET; 26s ago
       Docs: man:mariadbd(8)
``` 

```
[max@replication ~]$ sudo mysql_secure_installation

Enter current password for root (enter for none): Enter
Set root password? [Y/n] Y
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

### Configuration Database

```
[max@db ~]$ sudo nano /etc/my.cnf

#
# This group is read both both by the client and the server
# use it for options that affect everything
#
[client-server]

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
[mysqld]
bind-address=10.102.1.12
server-id=1
log_bin=mysql-bin
binlog-format=ROW
```

```
[max@db ~]$ sudo systemctl restart mariadb
``` 

```
[max@db ~]$ sudo mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 4
Server version: 10.5.16-MariaDB-log MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE USER 'mdoub'@'%' IDENTIFIED BY 'pewpewpew';
Query OK, 0 rows affected (0.008 sec)

MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO 'mdoub'@'%';
Query OK, 0 rows affected (0.006 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> Ctrl-C -- exit!
Aborted
[max@db ~]$ sudo mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
Server version: 10.5.16-MariaDB-log MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> STOP SLAVE;
Query OK, 0 rows affected, 1 warning (0.000 sec)

MariaDB [(none)]> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      769 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.000 sec)

MariaDB [(none)]> UNLOCK TABLES;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> Ctrl-C -- exit!
Aborted
```

```
[max@db ~]$ scp nextcloud.sql max@10.102.1.14:~/
The authenticity of host '10.102.1.14 (10.102.1.14)' can't be established.
ED25519 key fingerprint is SHA256:GdViisKeU1gpX9GuenydQcjOaEI4eMfSiIhS9f4HzLg.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.102.1.14' (ED25519) to the list of known hosts.
max@10.102.1.14's password:
nextcloud.sql                                               100%  196KB  34.7MB/s   00:00
```

### Configuration Database Replication

```
[max@replication ~]$ sudo nano /etc/my.cnf

#
# This group is read both both by the client and the server
# use it for options that affect everything
#
[client-server]

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
[mysqld]
bind-address=10.102.1.14
server-id=2
binlog-format=ROW
```

```
[max@replication ~]$ sudo mysql -u root -p nextcloud<nextcloud.sql

[max@replication ~]$ sudo mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 10
Server version: 10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> STOP SLAVE;
Query OK, 0 rows affected (0.005 sec)

MariaDB [(none)]> CHANGE MASTER TO MASTER_HOST = '10.102.1.12', MASTER_USER = 'mdoub', MASTER_PASSWORD = 'pewpewpew', MASTER_LOG_FILE = 'mysql-bin.000001', MASTER_LOG_POS = 284873;
Query OK, 0 rows affected (0.013 sec)

MariaDB [(none)]> START SLAVE;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> SHOW SLAVE STATUS
    -> \G
*************************** 1. row ***************************
                Slave_IO_State: Waiting for master to send event
                   Master_Host: 10.102.1.12
                   Master_User: mdoub
                   Master_Port: 3306
                 Connect_Retry: 60
               Master_Log_File: mysql-bin.000001
``` 

### Test

```
[max@db ~]$ sudo mysql -u root -p
[sudo] password for max:
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 10.5.16-MariaDB-log MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database database_test;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> Ctrl-C -- exit!
Aborted
``` 

```
[max@replication ~]$ sudo mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| database_test      |
+--------------------+
4 rows in set (0.001 sec)

MariaDB [(none)]> Ctrl-C -- exit!
Aborted
``` 




