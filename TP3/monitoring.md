# TP3 : Amélioration de la solution NextCloud - Maxim Doublait B2B

## Module 5 : Monitoring

Dans ce sujet on va installer un outil plutôt clé en main pour mettre en place un monitoring simple de nos machines.
L'outil qu'on va utiliser est Netdata.
➜ Je vous laisse suivre la doc pour le mettre en place ou ce genre de lien. Vous n'avez pas besoin d'utiliser le "Netdata Cloud" machin truc. Faites simplement une install locale.
Installez-le sur web.tp2.linux et db.tp2.linux.
Une fois en place, Netdata déploie une interface un Web pour avoir moult stats en temps réel, utilisez une commande ss pour repérer sur quel port il tourne.
Utilisez votre navigateur pour visiter l'interface web de Netdata http://<IP_VM>:<PORT_NETDATA>.
➜ Configurer Netdata pour qu'il vous envoie des alertes dans un salon Discord dédié en cas de soucis
➜ Vérifier que les alertes fonctionnent en surchargeant volontairement la machine par exemple (effectuez des stress tests de RAM et CPU, ou remplissez le disque volontairement par exemple)

``` 
[max@db ~]$ sudo dnf update
[max@db ~]$ sudo dnf install epel-release -y
Complete!
[max@db ~]$ sudo dnf install wget
Complete!
[max@db ~]$ wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh
```

```
[max@db ~]$ sudo systemctl start netdata
[max@db ~]$ sudo systemctl enable netdata
[max@db ~]$ sudo systemctl status netdata
● netdata.service - Real time performance monitoring
     Loaded: loaded (/usr/lib/systemd/system/netdata.service; enabled; vendor preset: disable>
     Active: active (running) since Wed 2022-11-16 00:29:13 CET; 2min 57s ago
```

```
PS C:\Users\mdoub\Desktop> curl 10.102.1.12:19999


StatusCode        : 200
StatusDescription : OK
Content           : <!doctype html><html lang="en"><head><title>netdata dashboard</title><meta name="application-name" content="netdata"><meta
                    http-equiv="Content-Type" content="text/html; charset=utf-8"/><meta charset="...
``` 

```
[max@db ~]$ sudo ss -alpnt | grep netdata
LISTEN 0      4096       127.0.0.1:8125       0.0.0.0:*    users:(("netdata",pid=46007,fd=55))
LISTEN 0      4096         0.0.0.0:19999      0.0.0.0:*    users:(("netdata",pid=46007,fd=6))
LISTEN 0      4096           [::1]:8125          [::]:*    users:(("netdata",pid=46007,fd=34))
LISTEN 0      4096            [::]:19999         [::]:*    users:(("netdata",pid=46007,fd=7))
``` 

