# TP3 : Amélioration de la solution NextCloud - Maxim Doublait B2B

## Module 7 : Fail2ban

```
[max@db ~]$ sudo dnf install epel-release -y
Package epel-release-9-4.el9.noarch is already installed.
Dependencies resolved.
Nothing to do.
Complete!

[max@db ~]$ sudo dnf install fail2ban -y
Installed:
  esmtp-1.2-19.el9.x86_64                        fail2ban-1.0.1-2.el9.noarch
  fail2ban-firewalld-1.0.1-2.el9.noarch          fail2ban-sendmail-1.0.1-2.el9.noarch
  fail2ban-server-1.0.1-2.el9.noarch             libesmtp-1.0.6-24.el9.x86_64
  liblockfile-1.14-9.el9.x86_64                  python3-systemd-234-18.el9.x86_64

Complete!
``` 

```
[max@db ~]$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
``` 

```
[max@db ~]$ sudo mv /etc/fail2ban/jail.d/00-firewalld.conf /etc/fail2ban/jail.d/00-firewalld.local

[max@db ~]$ sudo nano /etc/fail2ban/jail.d/sshd.local   
[sshd]
enabled = true
bantime = 1d
findtime = 1m
maxretry = 3

[max@db ~]$ sudo systemctl restart fail2ban
[max@db ~]$ sudo systemctl start fail2ban
[max@db ~]$ sudo systemctl enable fail2ban
Created symlink /etc/systemd/system/multi-user.target.wants/fail2ban.service → /usr/lib/systemd/system/fail2ban.service.
```
```
[max@db ~]$ sudo systemctl status fail2ban
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/usr/lib/systemd/system/fail2ban.service; enabled; vendor preset: disabl>
     Active: active (running) since Wed 2022-11-16 02:05:44 CET; 7min ago
       Docs: man:fail2ban(1)
```

```
[max@localhost ~]$ ssh max@10.102.1.12
The authenticity of host '10.102.1.12 (10.102.1.12)' can't be established.
ED25519 key fingerprint is SHA256:GdViisKeU1gpX9GuenydQcjOaEI4eMfSiIhS9f4HzLg.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.102.1.12' (ED25519) to the list of known hosts.
max@10.102.1.12's password:
Permission denied, please try again.
max@10.102.1.12's password:
Permission denied, please try again.
max@10.102.1.12's password:
max@10.102.1.12: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
[max@localhost ~]$ ssh max@10.102.1.12
ssh: connect to host 10.102.1.12 port 22: Connection refused
``` 

```
[max@db ~]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     3
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 1
   |- Total banned:     1
   `- Banned IP list:   10.102.1.11
``` 

```
[max@db ~]$ sudo fail2ban-client unban 10.102.1.11
1

[max@db ~]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     3
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 0
   |- Total banned:     1
   `- Banned IP list:
``` 

```
[max@localhost ~]$ ssh max@10.102.1.12
max@10.102.1.12's password:
Last failed login: Wed Nov 16 02:14:53 CET 2022 from 10.102.1.11 on ssh:notty
There were 3 failed login attempts since the last successful login.
Last login: Tue Nov 15 12:10:30 2022 from 10.102.1.1
``` 

