# TP3 : Amélioration de la solution NextCloud - Maxim Doublait B2B

## Module 1 : Reverse Proxy

Un reverse proxy est donc une machine que l'on place devant un autre service afin d'accueillir les clients et servir d'intermédiaire entre le client et le service.

L'utilisation d'un reverse proxy peut apporter de nombreux bénéfices :

- décharger le service HTTP de devoir effectuer le chiffrement HTTPS (coûteux en performances)
- répartir la charge entre plusieurs services
- effectuer de la mise en cache
- fournir un rempart solide entre un hacker potentiel et le service et les données importantes
- servir de point d'entrée unique pour accéder à plusieurs services web


## Sommaire

- [Module 1 : Reverse Proxy](#module-1--reverse-proxy)
  - [Sommaire](#sommaire)
- [I. Intro](#i-intro)
- [II. Setup](#ii-setup)
- [III. HTTPS](#iii-https)

# I. Intro

# II. Setup

🖥️ **VM `proxy.tp3.linux`**

**N'oubliez pas de dérouler la [📝**checklist**📝](#checklist).**

➜ **On utilisera NGINX comme reverse proxy**

- installer le paquet `nginx`
```
[max@proxy ~]$ sudo dnf install nginx
Installed:
  nginx-1:1.20.1-10.el9.x86_64                 nginx-filesystem-1:1.20.1-10.el9.noarch
  rocky-logos-httpd-90.11-1.el9.noarch

Complete!
```
- démarrer le service `nginx`
```
[max@proxy ~]$ sudo systemctl start nginx
``` 
- utiliser la commande `ss` pour repérer le port sur lequel NGINX écoute
```
[max@proxy ~]$ sudo ss -alpnt
State       Recv-Q      Send-Q           Local Address:Port            Peer Address:Port      Process
LISTEN      0           511                    0.0.0.0:80                   0.0.0.0:*          users:(("nginx",pid=10646,fd=6),("nginx",pid=10645,fd=6))
LISTEN      0           128                    0.0.0.0:22                   0.0.0.0:*          users:(("sshd",pid=683,fd=3))
LISTEN      0           511                       [::]:80                      [::]:*          users:(("nginx",pid=10646,fd=7),("nginx",pid=10645,fd=7))
LISTEN      0           128                       [::]:22                      [::]:*          users:(("sshd",pid=683,fd=4))
```
- ouvrir un port dans le firewall pour autoriser le trafic vers NGINX
```
[max@proxy ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
[max@proxy ~]$ sudo firewall-cmd --reload
success
```
- utiliser une commande `ps -ef` pour déterminer sous quel utilisateur tourne NGINX
```
[max@proxy ~]$ ps -ef | grep nginx
root       10645       1  0 12:25 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx      10646   10645  0 12:25 ?        00:00:00 nginx: worker process
max        10692     820  0 12:34 pts/0    00:00:00 grep --color=auto nginx
``` 
- vérifier que le page d'accueil NGINX est disponible en faisant une requête HTTP sur le port 80 de la machine
```
[max@proxy ~]$ curl 10.102.1.13:80
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
```

➜ **Configurer NGINX**

- nous ce qu'on veut, c'pas une page d'accueil moche, c'est que NGINX agisse comme un reverse proxy entre les clients et notre serveur Web
- deux choses à faire :
  - créer un fichier de configuration NGINX
    - la conf est dans `/etc/nginx`
    - procédez comme pour Apache : repérez les fichiers inclus par le fichier de conf principal, et créez votre fichier de conf en conséquence
```
[max@proxy ~]$ sudo nano /etc/nginx/conf.d/nginx2.conf                                         
server {
    # On indique le nom que client va saisir pour accéder au service
    # Pas d'erreur ici, c'est bien le nom de web, et pas de proxy qu'on veut ici !
    server_name web.tp2.linux;

    # Port d'écoute de NGINX
    listen 80;

    location / {
        # On définit des headers HTTP pour que le proxying se passe bien
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-Host $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

        # On définit la cible du proxying
        proxy_pass http://10.102.1.11:80;
    }

    # Deux sections location recommandés par la doc NextCloud
    location /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
}
``` 
  - NextCloud est un peu exigeant, et il demande à être informé si on le met derrière un reverse proxy
    - y'a donc un fichier de conf NextCloud à modifier
    - c'est un fichier appelé `config.php`

```
[max@localhost ~]$ sudo nano /var/www/tp2_nextcloud/config/config.php
<?php
$CONFIG = array (
  'instanceid' => 'ocqglnv0qzhq',
  'passwordsalt' => 'VYbp4DAnuICPXhqYzaJVboGUS49mxt',
  'secret' => '62gdk2WvRb5LTXf6nCZ9SNo2C7+B59CipTsWmlXs+V8Q2GpC',
  'trusted_domains' =>
  array (
    0 => 'web.tp2.linux',
  ),
  'datadirectory' => '/var/www/tp2_nextcloud/data',
  'dbtype' => 'mysql',
  'version' => '25.0.0.15',
  'overwrite.cli.url' => 'http://web.tp2.linux',
  'dbname' => 'nextcloud',
  'dbhost' => '10.102.1.12:3306',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'nextcloud',
  'dbpassword' => 'pewpewpew',
  'installed' => true,
  'trusted_proxies' =>
  array (
    0 => '10.102.1.13:80'
  ),
  'overwritehost' => 'web.tp2.linux',
  'overwriteprotocol' => 'http',
);
```

Référez-vous à monsieur Google pour tout ça :)


➜ **Modifier votre fichier `hosts` de VOTRE PC**

- pour que le service soit joignable avec le nom `web.tp2.linux`
- c'est à dire que `web.tp2.linux` doit pointer vers l'IP de `proxy.tp3.linux`
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
10.102.1.13   web.tp2.linux
::1			localhost
``` 
- autrement dit, pour votre PC :
  - `web.tp2.linux` pointe vers l'IP du reverse proxy
  - `proxy.tp3.linux` ne pointe vers rien
  - faire un `ping` manuel vers l'IP de `proxy.tp3.linux` fonctionne
  - faire un `ping` manuel vers l'IP de `web.tp2.linux` ne fonctionne pas
  - taper `http://web.tp2.linux` permet d'accéder au site (en passant de façon transparente par l'IP du proxy)

> Oui vous ne rêvez pas : le nom d'une machine donnée pointe vers l'IP d'une autre ! Ici, on fait juste en sorte qu'un certain nom permette d'accéder au service, sans se soucier de qui porte réellement ce nom.

✨ **Bonus** : rendre le serveur `web.tp2.linux` injoignable sauf depuis l'IP du reverse proxy. En effet, les clients ne doivent pas joindre en direct le serveur web : notre reverse proxy est là pour servir de serveur frontal.

# III. HTTPS

Le but de cette section est de permettre une connexion chiffrée lorsqu'un client se connecte. Avoir le ptit HTTPS :)

Le principe :

- on génère une paire de clés sur le serveur `proxy.tp3.linux`
  - une des deux clés sera la clé privée : elle restera sur le serveur et ne bougera jamais
  - l'autre est la clé publique : elle sera stockée dans un fichier appelé *certificat*
    - le *certificat* est donné à chaque client qui se connecte au site
```
[max@proxy ~]$ openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt
Common Name (eg, your name or your server's hostname) []:web.tp2.linux
``` 

```
[max@proxy ~]$ sudo mv server.crt /etc/pki/tls/certs/
[max@proxy ~]$ sudo mv server.key /etc/pki/tls/private/
[max@proxy ~]$ sudo firewall-cmd --add-port=443/tcp --permanent
[max@proxy ~]$ sudo firewall-cmd --reload
``` 

- on ajuste la conf NGINX
  - on lui indique le chemin vers le certificat et la clé privée afin qu'il puisse les utiliser pour chiffrer le trafic
  - on lui demande d'écouter sur le port convetionnel pour HTTPS : 443 en TCP

``` 
[max@proxy ~]$ sudo nano /etc/nginx/conf.d/nginx2.conf
server {
    # On indique le nom que client va saisir pour accéder au service
    # Pas d'erreur ici, c'est bien le nom de web, et pas de proxy qu'on veut ici !
    server_name web.tp2.linux;


    # Port d'écoute de NGINX
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    ssl_certificate     /etc/pki/tls/certs/server.crt;
    ssl_certificate_key   /etc/pki/tls/private/server.key;

    location / {
        # On définit des headers HTTP pour que le proxying se passe bien
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-Host $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

        # On définit la cible du proxying
        proxy_pass http://10.102.1.11:80;
    }

    # Deux sections location recommandés par la doc NextCloud
    location /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
}
server {
    listen 80 default_server;
    server_name _;
    return 301 https://$host$request_uri;
}
```

```
[max@localhost ~]$ sudo nano /var/www/tp2_nextcloud/config/config.php
<?php
$CONFIG = array (
  'instanceid' => 'ocqglnv0qzhq',
  'passwordsalt' => 'VYbp4DAnuICPXhqYzaJVboGUS49mxt',
  'secret' => '62gdk2WvRb5LTXf6nCZ9SNo2C7+B59CipTsWmlXs+V8Q2GpC',
  'trusted_domains' =>
  array (
    0 => 'web.tp2.linux',
  ),
  'datadirectory' => '/var/www/tp2_nextcloud/data',
  'dbtype' => 'mysql',
  'version' => '25.0.0.15',
  'overwrite.cli.url' => 'https://web.tp2.linux',
  'overwriteprotocol' => '
  'dbname' => 'nextcloud',
  'dbhost' => '10.102.1.12:3306',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'nextcloud',
  'dbpassword' => 'pewpewpew',
  'installed' => true,
  'trusted_proxies' =>
  array (
    0 => '10.102.1.13:80'
  ),
  'overwritehost' => 'web.tp2.linux',
  'overwriteprotocol' => 'http',
);
``` 

```
[max@localhost ~]$ sudo systemctl restart httpd
[max@proxy ~]$ sudo systemctl restart nginx
``` 

Je vous laisse Google vous-mêmes "nginx reverse proxy nextcloud" ou ce genre de chose :)
