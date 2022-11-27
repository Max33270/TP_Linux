# TP4 : Conteneurs - Maxim Doublait B2B

Dans ce TP on va aborder plusieurs points autour de la conteneurisation : 

- Docker et son empreinte sur le système
- Manipulation d'images
- `docker-compose`


# Sommaire

- [TP4 : Conteneurs](#tp4--conteneurs)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
  - [Checklist](#checklist)
- [I. Docker](#i-docker)
  - [1. Install](#1-install)
  - [2. Vérifier l'install](#2-vérifier-linstall)
  - [3. Lancement de conteneurs](#3-lancement-de-conteneurs)
- [II. Images](#ii-images)
- [III. `docker-compose`](#iii-docker-compose)
  - [1. Intro](#1-intro)
  - [2. Make your own meow](#2-make-your-own-meow)

# 0. Prérequis

➜ Machines Rocky Linux

➜ Un unique host-only côté VBox, ça suffira. **L'adresse du réseau host-only sera `10.104.1.0/24`.**

➜ Chaque **création de machines** sera indiquée par **l'emoji 🖥️ suivi du nom de la machine**

➜ Si je veux **un fichier dans le rendu**, il y aura l'**emoji 📁 avec le nom du fichier voulu**. Le fichier devra être livré tel quel dans le dépôt git, ou dans le corps du rendu Markdown si c'est lisible et correctement formaté.


## Checklist

A chaque machine déployée, vous **DEVREZ** vérifier la 📝**checklist**📝 :

- [x] IP locale, statique ou dynamique
- [x] hostname défini
- [x] firewall actif, qui ne laisse passer que le strict nécessaire
- [x] SSH fonctionnel avec un échange de clé
- [x] accès Internet (une route par défaut, une carte NAT c'est très bien)
- [x] résolution de nom
- [x] SELinux désactivé (vérifiez avec `sestatus`, voir [mémo install VM tout en bas](https://gitlab.com/it4lik/b2-reseau-2022/-/blob/main/cours/memo/install_vm.md#4-pr%C3%A9parer-la-vm-au-clonage))

**Les éléments de la 📝checklist📝 sont STRICTEMENT OBLIGATOIRES à réaliser mais ne doivent PAS figurer dans le rendu.**

# I. Docker

🖥️ Machine **docker1.tp4.linux**

## 1. Install

🌞 **Installer Docker sur la machine**

- en suivant [la doc officielle](https://docs.docker.com/engine/install/)

```
[max@docker1 ~]$ sudo dnf install -y yum-utils 
Installed:
  yum-utils-4.0.24-4.el9_0.noarch

Complete!

[max@docker1 ~]$ sudo yum-config-manager --add-repo https://download.docker.com/lin
ux/centos/docker-ce.repo
Adding repo from: https://download.docker.com/linux/centos/docker-ce.repo

[max@docker1 ~]$ sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
Installed:
  checkpolicy-3.3-1.el9.x86_64
  container-selinux-3:2.188.0-1.el9_0.noarch
  containerd.io-1.6.10-3.1.el9.x86_64
  docker-ce-3:20.10.21-3.el9.x86_64
  docker-ce-cli-1:20.10.21-3.el9.x86_64
  docker-ce-rootless-extras-20.10.21-3.el9.x86_64
  docker-compose-plugin-2.12.2-3.el9.x86_64
  docker-scan-plugin-0.21.0-3.el9.x86_64
  fuse-common-3.10.2-5.el9.0.1.x86_64
  fuse-overlayfs-1.9-1.el9_0.x86_64
  fuse3-3.10.2-5.el9.0.1.x86_64
  fuse3-libs-3.10.2-5.el9.0.1.x86_64
  libslirp-4.4.0-7.el9.x86_64
  policycoreutils-python-utils-3.3-6.el9_0.noarch
  python3-audit-3.0.7-101.el9_0.2.x86_64
  python3-libsemanage-3.3-2.el9.x86_64
  python3-policycoreutils-3.3-6.el9_0.noarch
  python3-setools-4.4.0-4.el9.x86_64
  python3-setuptools-53.0.0-10.el9.noarch
  slirp4netns-1.2.0-2.el9_0.x86_64
  tar-2:1.34-3.el9.x86_64

Complete!
``` 

- démarrer le service `docker` avec une commande `systemctl`
```
[max@docker1 ~]$ sudo systemctl start docker
``` 
- ajouter votre utilisateur au groupe `docker`
  - cela permet d'utiliser Docker sans avoir besoin de l'identité de `root`
  - avec la commande : `sudo usermod -aG docker $(whoami)`
```
[max@docker1 ~]$ sudo usermod -aG docker max
``` 
  - déconnectez-vous puis relancez une session pour que le changement prenne effet
```
[max@docker1 ~]$ grep docker /etc/group
docker:x:991:max
``` 



## 2. Vérifier l'install

➜ **Vérifiez que Docker est actif est disponible en essayant quelques commandes usuelles :**

```bash
# Info sur l'install actuelle de Docker
$ docker info

# Liste des conteneurs actifs
$ docker ps
# Liste de tous les conteneurs
$ docker ps -a

# Liste des images disponibles localement
$ docker images

# Lancer un conteneur debian
$ docker run debian
$ docker run -d debian sleep 99999
$ docker run -it debian bash

# Consulter les logs d'un conteneur
$ docker ps # on repère l'ID/le nom du conteneur voulu
$ docker logs <ID_OR_NAME>
$ docker logs -f <ID_OR_NAME> # suit l'arrivée des logs en temps réel

# Exécuter un processus dans un conteneur actif
$ docker ps # on repère l'ID/le nom du conteneur voulu
$ docker exec <ID_OR_NAME> <COMMAND>
$ docker exec <ID_OR_NAME> ls
$ docker exec -it <ID_OR_NAME> bash # permet de récupérer un shell bash dans le conteneur ciblé
```

```
[max@docker1 ~]$ docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Docker Buildx (Docker Inc., v0.9.1-docker)
  compose: Docker Compose (Docker Inc., v2.12.2)
  scan: Docker Scan (Docker Inc., v0.21.0)

[max@docker1 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

[max@docker1 ~]$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
7ae78cafb519   debian        "bash"     22 seconds ago   Exited (0) 21 seconds ago             trusting_bassi
5502d4678d2a   hello-world   "/hello"   7 minutes ago    Exited (0) 7 minutes ago              goofy_ptolemy

[max@docker1 ~]$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
debian        latest    c31f65dd4cc9   9 days ago      124MB
hello-world   latest    feb5d9fea6a5   14 months ago   13.3kB

[max@docker1 ~]$ docker run debian

[max@docker1 ~]$ docker run -d debian sleep 99999
c58a99506874f0394e2e75a826c55e93c65b1a4326a64b77caff8b9529033a26

[max@docker1 ~]$ docker run -it debian bash
root@3b30fe29d81c:/# exit
exit

[max@docker1 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND         CREATED          STATUS          PORTS     NAMES
c58a99506874   debian    "sleep 99999"   35 seconds ago   Up 34 seconds             hopeful_edison

[max@docker1 ~]$ docker logs c58a99506874

[max@docker1 ~]$ docker logs hopeful_edison

[max@docker1 ~]$ docker logs -f hopeful_edison
^C

[max@docker1 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND         CREATED         STATUS         PORTS     NAMES
c58a99506874   debian    "sleep 99999"   2 minutes ago   Up 2 minutes             hopeful_edison

[max@docker1 ~]$ docker exec c58a99506874 ls
bin
boot
dev
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

[max@docker1 ~]$ docker exec -it c58a99506874 bash
root@c58a99506874:/# exit
exit
```

➜ **Explorer un peu le help**, si c'est pas le man :

```bash
$ docker --help
$ docker run --help
$ man docker
```

## 3. Lancement de conteneurs

La commande pour lancer des conteneurs est `docker run`.

Certaines options sont très souvent utilisées :

```bash
# L'option --name permet de définir un nom pour le conteneur
$ docker run --name web nginx

# L'option -d permet de lancer un conteneur en tâche de fond
$ docker run --name web -d nginx

# L'option -v permet de partager un dossier/un fichier entre l'hôte et le conteneur
$ docker run --name web -d -v /path/to/html:/usr/share/nginx/html nginx

# L'option -p permet de partager un port entre l'hôte et le conteneur
$ docker run --name web -d -v /path/to/html:/usr/share/nginx/html -p 8888:80 nginx
# Dans l'exemple ci-dessus, le port 8888 de l'hôte est partagé vers le port 80 du conteneur
```

🌞 **Utiliser la commande `docker run`**

- lancer un conteneur `nginx`
  - l'app NGINX doit avoir un fichier de conf personnalisé
  - l'app NGINX doit servir un fichier `index.html` personnalisé
  - l'application doit être joignable grâce à un partage de ports
  - vous limiterez l'utilisation de la RAM et du CPU de ce conteneur
  - le conteneur devra avoir un nom
  - le processus exécuté par le conteneur doit être un utilisateur de votre choix (pas `root`)

> Tout se fait avec des options de la commande `docker run`.

```
[max@docker1 ~]$ docker run --name web -d nginx
4f4c34538ab958a02bb36d07cb29939768ed0698a1e6b3a38602b3168d08a91b
``` 

```
[max@docker1 ~]$ sudo cat /home/max/nginx2.conf
 server {

    listen 8888;
    root /var/www/toto
}
``` 

```
[max@docker1 ~]$ cat index.html
<!DOCTYPE html>
<html lang="us">
<head>
    <meta charset="UTF-8">
    <title> Max's page :) </title>
</head>
<body>
    <h1>welcome to Max's site</h1>
    <p> in html </p>
</body>
</html>
```

```
[max@docker1 ~]$ sudo firewall-cmd --add-port=8888/tcp --permanent
success
[max@docker1 ~]$ sudo firewall-cmd --reload
success
``` 

```
[max@docker1 ~]$ docker run --rm -p 33:8888 -v /home/max/nginx2.conf:/etc/nginx/conf.d/nginx2.conf -v /home/max/index.html:/var/www/toto/index.html --name web -m="1g" --cpus="1.0"  nginx

/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2022/11/24 14:38:08 [notice] 1#1: using the "epoll" event method
2022/11/24 14:38:08 [notice] 1#1: nginx/1.23.2
2022/11/24 14:38:08 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
2022/11/24 14:38:08 [notice] 1#1: OS: Linux 5.14.0-70.26.1.el9_0.x86_64
2022/11/24 14:38:08 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1073741816:1073741816
2022/11/24 14:38:08 [notice] 1#1: start worker processes
2022/11/24 14:38:08 [notice] 1#1: start worker process 28
10.104.1.1 - - [24/Nov/2022:14:38:19 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36" "-"
```
```
C:\Users\mdoub>curl 10.104.1.0:33
<!DOCTYPE html>
<html lang="us">
<head>
    <meta charset="UTF-8">
    <title> Max's page :) </title>
</head>
<body>
    <h1>welcome to Max's site</h1>
    <p> in html </p>
</body>
</html>
``` 

# II. Images

La construction d'image avec Docker est basée sur l'utilisation de fichiers `Dockerfile`.

🌞 **Construire votre propre image**

- image de base
  - une image du Docker Hub
  - digne de confiance
  - qui ne porte aucune application par défaut
- vous ajouterez
  - mise à jour du système
  - installation de Apache
  - page d'accueil Apache HTML personnalisée

📁 **`Dockerfile`**

```
[max@docker1 ~]$ sudo mkdir docker_workspace

[max@docker1 ~]$ cd docker_workspace/

[max@docker1 docker_workspace]$ cat Dockerfile
FROM debian

RUN apt update -y && \
    apt install -y apache2

ADD index.html /var/www/html

ADD apache2Conf.conf /etc/apache2/apache2.conf

RUN mkdir /etc/apache2/logs

CMD [ "/usr/lib/apache2", "-g", "daemon off;" ]


[max@docker1 docker_workspace]$ ls
Dockerfile
``` 
```
[max@docker1 docker_workspace]$ docker build . -t my_own_apache
Successfully built dcb68193300f
Successfully tagged my_own_apache:latest

[max@docker1 docker_workspace]$ docker images
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
my_own_apache   latest    dcb68193300f   43 seconds ago   253MB
```
```
[max@docker1 docker_workspace]$ cat index.html
<!DOCTYPE html>
<html>
<title>Apache</title>
<body>
<h1>Welcome to APache</h1>
</body>
</html>
``` 
```
[max@docker1 docker_workspace]$ cat apache2Conf.conf
# on définit un port sur lequel écouter
Listen 80
ServerName 10.104.1.O
# on charge certains modules Apache strictement nécessaires à son bon fonctionnement
LoadModule mpm_event_module "/usr/lib/apache2/modules/mod_mpm_event.so"
LoadModule dir_module "/usr/lib/apache2/modules/mod_dir.so"
LoadModule authz_core_module "/usr/lib/apache2/modules/mod_authz_core.so"

# on indique le nom du fichier HTML à charger par défaut
DirectoryIndex index.html
# on indique le chemin où se trouve notre site
DocumentRoot "/var/www/html/"

# quelques paramètres pour les logs
ErrorLog "logs/error.log"
LogLevel warn
```
```
[max@docker1 docker_workspace]$ docker build . -t my_own_apache
Successfully built ce7965f5899a
Successfully tagged my_own_apache:latest

[max@docker1 docker_workspace]$ docker run -p 8888:80 my_own_apache apache2 -DFOREGROUND
```

```
mdoub@MAX-PC MINGW64 /
$ curl 10.104.1.0:8888
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    95  100    95    0     0  35133      0 --:--:-- --:--:-- --:--:-- 95000<!DOCTYPE html>
<html>
<title>Apache</title>
<body>
<h1>Welcome to APache</h1>
</body>
</html>
```
# III. `docker-compose`

## 1. Intro

➜ **Installer `docker-compose` sur la machine**

- en suivant [la doc officielle](https://docs.docker.com/compose/install/)

`docker-compose` est un outil qui permet de lancer plusieurs conteneurs en une seule commande.

> En plus d'être pratique, il fournit des fonctionnalités additionnelles, liés au fait qu'il s'occupe à lui tout seul de lancer tous les conteneurs. On peut par exemple demander à un conteneur de ne s'allumer que lorsqu'un autre conteneur est devenu "healthy". Idéal pour lancer une application après sa base de données par exemple.


Le principe de fonctionnement de `docker-compose` :

- on écrit un fichier qui décrit les conteneurs voulus
  - c'est le `docker-compose.yml`
  - tout ce que vous écriviez sur la ligne `docker run` peut être écrit sous la forme d'un `docker-compose.yml`
- on se déplace dans le dossier qui contient le `docker-compose.yml`
- on peut utiliser les commandes `docker-compose` :

```bash
# Allumer les conteneurs définis dans le docker-compose.yml
$ docker-compose up
$ docker-compose up -d

# Eteindre
$ docker-compose down

# Explorer un peu le help, il y a d'autres commandes utiles
$ docker-compose --help
```

La syntaxe du fichier peut par exemple ressembler à :

```yml
version: "3.8"

services:
  db:
    image: mysql:5.7
    restart: always
    ports:
      - '3306:3306'
    volumes:
      - "./db/mysql_files:/var/lib/mysql"
    environment:
      MYSQL_ROOT_PASSWORD: beep
      MYSQL_DATABASE: bip
      MYSQL_USER: bap
      MYSQL_PASSWORD: boop

  nginx:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    restart: unless-stopped
```

> Pour connaître les variables d'environnement qu'on peut passer à un conteneur, comme `MYSQL_ROOT_PASSWORD` au dessus, il faut se rendre sur la doc de l'image en question, sur le Docker Hub par exemple.

## 2. Make your own meow

Pour cette partie, vous utiliserez une application à vous que vous avez sous la main.

N'importe quelle app fera le taff, un truc dév en cours, en temps perso, au taff, peu importe.

Peu importe le langage aussi ! Go, Python, PHP (désolé des gros mots), Node (j'ai déjà dit désolé pour les gros mots ?), ou autres.

🌞 **Conteneurisez votre application**

- créer un `Dockerfile` maison qui porte l'application
- créer un `docker-compose.yml` qui permet de lancer votre application
- vous préciserez dans le rendu les instructions pour lancer l'application
  - indiquer la commande `git clone`
  - le `cd` dans le bon dossier
  - la commande `docker build` pour build l'image
  - la commande `docker-compose` pour lancer le(s) conteneur(s)

📁 📁 `app/Dockerfile` et `app/docker-compose.yml`. Je veux un sous-dossier `app/` sur votre dépôt git avec ces deux fichiers dedans :)

### Pour lancer l'application : 
```
[max@docker1 ~]$ git clone https://github.com/Max33270/Golang-HangmanWeb.git

[max@localhost ~]$ cd app/

[max@localhost app]$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
hangman       latest    220b42d85404   12 minutes ago   892MB
debian        latest    c31f65dd4cc9   12 days ago      124MB
hello-world   latest    feb5d9fea6a5   14 months ago    13.3kB

[max@localhost main]$ docker build . -t hangman
Sending build context to Docker daemon  7.168kB
Step 1/6 : FROM debian
 ---> c31f65dd4cc9
Step 2/6 : RUN apt update -y
 ---> Using cache
 ---> 46628747b5c1
Step 3/6 : RUN apt install -y golang git
 ---> Using cache
 ---> af002502386e
Step 4/6 : RUN /usr/bin/git clone https://github.com/Max33270/Golang-HangmanWeb2.git /app
 ---> Using cache
 ---> 52b5592d5dfe
Step 5/6 : WORKDIR /app
 ---> Using cache
 ---> c1b4353d5605
Step 6/6 : CMD [ "/usr/bin/go", "run", "./main/main.go" ]
 ---> Running in 370dd36a3397
Removing intermediate container 370dd36a3397
 ---> fd55eafb76fb
Successfully built fd55eafb76fb
Successfully tagged hangman:latest

[max@localhost main]$ docker compose up
[+] Running 1/0
 ⠿ Container main-hangman-1  Recreated                                                   0.1s
Attaching to main-hangman-1
main-hangman-1  | Listening on:8080
``` 