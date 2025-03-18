# **Compte rendu : Personnalisation de Nginx avec Docker**
# Partie 1:
## **1. Installation et Configuration**

### **🌞 Installation de Docker sur la machine Azure**

installation de Docker via la doc:

```
GPG KEY DOCKER OFFICEL (trop sympa de fournir):
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

AJOUTER REPOSIT SUR APT
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```



## **2. Vérification de l'installation de Docker**
Les commandes suivantes ont été utilisées pour vérifier que Docker fonctionnait correctement :

```
docker --version
docker info
docker ps -a
docker images
```
Aucune erreur n'a été rencontrée et Docker a bien été installé et configuré.
### **Lancer un conteneur Nginx**
Pour lancer un conteneur Nginx avec la configuration par défaut, voici la commande de base :

```
docker run -d -p 9999:80 nginx   
```

Par défaut, Nginx écoute sur le port 80 et propose une page d'accueil.
Cette commande redirige le port 9999 de la machine hôte vers le port 80 du conteneur.


```
curl http://98.70.74.27:80
```
```
<!DOCTYPE html>
<html>
  <head><title>Welcome to nginx!</title></head>
  <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and working. Further configuration is required.</p>
  </body>
</html>
```

### **3. Lancement du Conteneur Nginx avec la Configuration Personnalisée**
### **🌞 Config :**

Ouverture du port 7777 en TCP et creation de mon fichier leNginx ou je vais stocker mon html perso et mon .conf

### **🌞 Commande Docker utilisée pour lancer le conteneur**
La commande suivante a permis de lancer le conteneur Nginx avec les éléments personnalisés :

```
docker run --name meow -d -p 7777:7777 -v ~/leNginx/html:/var/www/tp_docker -v ~/leNginx/conf/maconf.conf:/etc/nginx/conf.d/maconf.conf --memory=512m nginx
```

### **🌞Vérification de la configuration du fichier Nginx
Pour vérifier que le fichier de configuration était correctement monté dans le conteneur, la commande suivante a été exécutée :

```
docker exec -it meow cat /etc/nginx/conf.d/maconf.conf
Le contenu du fichier maconf.conf était bien celui attendu, configurant Nginx pour écouter sur le port 7777 et servir le fichier index.html situé dans le dossier /var/www/tp_docker.
```

Voici le contenu du fichier :

```
server {
    listen 7777;
    root /var/www/tp_docker;
    index index.html;
}
```

### **4. Vérification du site web**
### **🌞 Tester l'accès depuis le navigateur**
Le site a été accessible avec succès en utilisant l'URL suivante :
```
http://98.70.74.27:7777
```

### **🌞 Résultat attendu**
La page affichait le contenu personnalisé 


### **🌞 Tester avec curl**

```
curl http://98.70.74.27:7777
```
Ce qui nous donne:
```
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bienvenue sur AMWA</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background: linear-gradient(135deg, #1e3c72, #2a5298);
            color: white;
            font-family: Arial, sans-serif;
            text-align: center;
        }
        .container {
            animation: fadeIn 2s ease-in-out;
        }
        h1 {
            font-size: 3rem;
            margin-bottom: 10px;
        }
        .animated-text {
            font-size: 1.5rem;
            display: inline-block;
            overflow: hidden;
            border-right: 3px solid white;
            white-space: nowrap;
            letter-spacing: 2px;
            animation: typing 3s steps(20, end) forwards, blink 0.7s infinite;
        }
        @keyframes typing {
            from { width: 0; }
            to { width: 100%; }
        }
        @keyframes blink {
            50% { border-color: transparent; }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Bienvenue sur le Site <span style="color: #ffcc00;">AMWA</span></h1>
        <p class="animated-text">Une Expérience Unique Vous Attend...</p>
    </div>
</body>
</html>
```

# Partie 2:
Création du répertoire de travail
```
azureuser@TP:~$ sudo mkdir image_docker_tp
azureuser@TP:~$ cd image_docker_tp
```
Rédaction du Dockerfile
```
azureuser@TP:~/image_docker_tp$ sudo nano Dockerfile
```

Contenu du Dockerfile
```
FROM debian:latest

RUN apt update -y && apt install -y apache2

COPY apache2.conf /etc/apache2/apache2.conf

COPY index.html /var/www/html/index.html

EXPOSE 80

CMD [ "apache2", "-DFOREGROUND" ]
```
Contenu de apache2.conf
```
ServerName 98.70.74.27

Listen 80

LoadModule mpm_event_module "/usr/lib/apache2/modules/mod_mpm_event.so"
LoadModule dir_module "/usr/lib/apache2/modules/mod_dir.so"
LoadModule authz_core_module "/usr/lib/apache2/modules/mod_authz_core.so"

DirectoryIndex index.html

DocumentRoot "/var/www/html/"

ErrorLog "/var/log/apache2/error.log"
LogLevel warn
```
Ensuite faire un index.html personalisé

Puis construction de l'image docker
```docker build . -t image_docker_tp   
```
Lancement du Conteneur
```
azureuser@TP:~/image_docker_tp$ docker run -p 8080:80 image_docker_tp
```
Puis une verif avec Curl:
```
┌──(keyket㉿keyket (IP: 10.3.219.184))-[~]
└─$ curl http://98.70.74.27:8080  
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bienvenue sur mon serveur Apache rigolo !</title>
    <style>
        body {
            background-color: #f0f8ff;
            font-family: 'Comic Sans MS', cursive, sans-serif;
            text-align: center;
            color: #ff6347;
        }
        h1 {
            font-size: 3em;
            margin-top: 50px;
        }
        .cat {
            font-size: 5em;
            color: #ff69b4;
        }
        p {
            font-size: 1.5em;
        }
        button {
            background-color: #ff6347;
            color: white;
            font-size: 1.2em;
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        button:hover {
            background-color: #ff4500;
        }
    </style>
</head>
<body>
    <h1>Bienvenue sur le serveur Apache le plus fun !</h1>
    <div class="cat">🐱</div>
    <p>Le chat approuve la visite !</p>
    <p>Tu veux quelque chose de rigolo ? Clique sur le bouton ci-dessous !</p>
    <button onclick="alert('Wow, tu as trouvé le secret du serveur Apache rigolo ! 😸')">Clique-moi !</button>
</body>
</html>
```
# Partie 3:
Création du dossier et du fichier docker-compose.yml
```
azureuser@TP:~$ mkdir wikijs && cd wikijs
azureuser@TP:~$ sudo nano docker-compose.yml
```
docker-compose.yml
```
version: "3.8"

services:
  db:
    image: postgres:13
    restart: always
    environment:
      POSTGRES_DB: wikijs
      POSTGRES_USER: wikijs
      POSTGRES_PASSWORD: mysecretpassword
    volumes:
      - db-data:/var/lib/postgresql/data

  wikijs:
    image: requarks/wiki:latest
    restart: always
    depends_on:
      - db
    ports:
      - "80:3000"
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: mysecretpassword
      DB_NAME: wikijs

volumes:
  db-data:
```
Lancement de WikiJS
```
azureuser@TP:~/wikijs$ docker-compose up -d
```

Verif :

```
┌──(keyket㉿keyket (IP: 10.3.219.184))-[~]
└─$ curl http://98.70.74.27:80  
<!DOCTYPE html><html><head><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta charset="UTF-8"><meta name="viewport" content="user-scalable=yes, width=device-width, initial-scale=1, maximum-scale=5"><meta name="theme-color" content="#1976d2"><meta name="msapplication-TileColor" content="#1976d2"><meta name="msapplication-TileImage" content="/_assets/favicons/mstile-150x150.png"><title>Wiki.js Setup</title><link rel="apple-touch-icon" sizes="180x180" href="/_assets/favicons/apple-touch-icon.png"><link rel="icon" type="image/png" sizes="192x192" href="/_assets/favicons/android-chrome-192x192.png"><link rel="icon" type="image/png" sizes="32x32" href="/_assets/favicons/favicon-32x32.png"><link rel="icon" type="image/png" sizes="16x16" href="/_assets/favicons/favicon-16x16.png"><link rel="mask-icon" href="/_assets/favicons/safari-pinned-tab.svg" color="#1976d2"><link rel="manifest" href="/_assets/manifest.json"><script>var siteConfig = {"title":"Wiki.js"}
</script><link type="text/css" rel="stylesheet" href="/_assets/css/setup.22871ffac1b643eed4d9.css"><script type="text/javascript" src="/_assets/js/runtime.js?1738531300"></script><script type="text/javascript" src="/_assets/js/setup.js?1738531300"></script></head><body><div id="root"><setup wiki-version="2.5.306"></setup></div></body></html>  
```
ajout de l'utilisateur au group Docker
```
sudo usermod -aG docker $USER
```

Transfert des fichier de l'app
```
scp -r "/home/keyket/Desktop/ledoss" azureuser@98.70.74.27:~/meow

```
Créer le Dockerfile pour l’application
```
azureuser@TP:~/meow$ nano Dockerfile
```
Créer le fichier docker-compose.yml
```
azureuser@TP:~/meow$ nano docker-compose.yml
```
Lancer l'app:
```
azureuser@TP:~/meow$ docker compose up --build -d
```
Ptit Curl:
```
┌──(keyket㉿keyket (IP: 10.3.219.184))-[~]
└─$ curl http://98.70.74.27:8888                       
<h1>Add key</h1>
<form action="/add" method = "POST">

Key:
<input type="text" name="key" >

Value:
<input type="text" name="value" >

<input type="submit" value="Submit">
</form>

<h1>Check key</h1>
<form action="/get" method = "POST">

Key:
<input type="text" name="key" >
<input type="submit" value="Submit">
</form>
```


# Partie 4:
Verif de docker dans le groupe:
```
azureuser@TipTop:~$ groups
azureuser adm dialout cdrom floppy sudo audio dip video plugdev netdev lxd docker
```
Lancer un conteneur Docker en tant que root:
```

azureuser@TipTop:~$ docker run --rm -it --privileged alpine sh
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
f18232174bc9: Pull complete 
Digest: sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
Status: Downloaded newer image for alpine:latest
/ # ls
bin    etc    lib    mnt    proc   run    srv    tmp    var
dev    home   media  opt    root   sbin   sys    usr
/ # cat /etc/shadow
root:*::0:::::
bin:!::0:::::
daemon:!::0:::::
lp:!::0:::::
sync:!::0:::::
shutdown:!::0:::::
halt:!::0:::::
mail:!::0:::::
news:!::0:::::
uucp:!::0:::::
cron:!::0:::::
ftp:!::0:::::
sshd:!::0:::::
games:!::0:::::
ntp:!::0:::::
guest:!::0:::::
nobody:!::0:::::
/ # 
```

### Scanner les vulnérabilités avec Trivy:


##### Scan vuln image nginx :

Scan de monApache :
```
azureuser@TipTop:~$ trivy image monapache:latest
2025-03-17T11:41:45Z    INFO    [vulndb] Need to update DB
2025-03-17T11:41:45Z    INFO    [vulndb] Downloading vulnerability DB...
2025-03-17T11:41:45Z    INFO    [vulndb] Downloading artifact...        repo="mirror.gcr.io/aquasec/trivy-db:2"
60.97 MiB / 60.97 MiB [-------------------------] 100.00% 4.02 MiB p/s 15s
2025-03-17T11:42:01Z    INFO    [vulndb] Artifact successfully downloadedrepo="mirror.gcr.io/aquasec/trivy-db:2"
2025-03-17T11:42:01Z    INFO    [vuln] Vulnerability scanning is enabled
2025-03-17T11:42:01Z    INFO    [secret] Secret scanning is enabled
2025-03-17T11:42:01Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-03-17T11:42:01Z    INFO    [secret] Please see also https://trivy.dev/v0.60/docs/scanner/secret#recommendation for faster secret detection
2025-03-17T11:42:33Z    INFO    Detected OS     family="debian" version="12.9"
2025-03-17T11:42:33Z    INFO    [debian] Detecting vulnerabilities...   os_version="12" pkg_num=136
2025-03-17T11:42:33Z    INFO    Number of language-specific files       num=0
2025-03-17T11:42:33Z    WARN    Using severities from other vendors for some vulnerabilities. Read https://trivy.dev/v0.60/docs/scanner/vulnerability#severity-selection for details.

Report Summary

┌────────────────────────────────────────┬────────┬─────────────────┬─────────┐
│                 Target                 │  Type  │ Vulnerabilities │ Secrets │
├────────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ monapache:latest (debian 12.9)         │ debian │       174       │    -    │
├────────────────────────────────────────┼────────┼─────────────────┼─────────┤
│ /etc/ssl/private/ssl-cert-snakeoil.key │  text  │        -        │    1    │
└────────────────────────────────────────┴────────┴─────────────────┴─────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)
```
##### Scan vuln image nginx :

```
azureuser@TipTop:~$ trivy image nginx:latest
2025-03-17T11:43:27Z    INFO    [vuln] Vulnerability scanning is enabled
2025-03-17T11:43:27Z    INFO    [secret] Secret scanning is enabled
2025-03-17T11:43:27Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-03-17T11:43:27Z    INFO    [secret] Please see also https://trivy.dev/v0.60/docs/scanner/secret#recommendation for faster secret detection
2025-03-17T11:43:53Z    INFO    [javadb] Downloading Java DB...
2025-03-17T11:43:53Z    INFO    [javadb] Downloading artifact...        repo="mirror.gcr.io/aquasec/trivy-java-db:1"
697.86 MiB / 697.86 MiB [---------------------] 100.00% 9.49 MiB p/s 1m14s
2025-03-17T11:45:08Z    INFO    [javadb] Artifact successfully downloadedrepo="mirror.gcr.io/aquasec/trivy-java-db:1"
2025-03-17T11:45:08Z    INFO    [javadb] Java DB is cached for 3 days. If you want to update the database more frequently, "trivy clean --java-db" command clears the DB cache.
2025-03-17T11:45:08Z    INFO    Detected OS     family="debian" version="12.9"
2025-03-17T11:45:08Z    INFO    [debian] Detecting vulnerabilities...   os_version="12" pkg_num=149
2025-03-17T11:45:08Z    INFO    Number of language-specific files       num=0
2025-03-17T11:45:08Z    WARN    Using severities from other vendors for some vulnerabilities. Read https://trivy.dev/v0.60/docs/scanner/vulnerability#severity-selection for details.

Report Summary

┌────────────────────────────┬────────┬─────────────────┬─────────┐
│           Target           │  Type  │ Vulnerabilities │ Secrets │
├────────────────────────────┼────────┼─────────────────┼─────────┤
│ nginx:latest (debian 12.9) │ debian │       156       │    -    │
└────────────────────────────┴────────┴─────────────────┴─────────┘
```
#### Scan vuln BDD Wikijs


```
azureuser@TipTop:~$ trivy image postgres:13
2025-03-17T11:46:35Z    INFO    [vuln] Vulnerability scanning is enabled
2025-03-17T11:46:35Z    INFO    [secret] Secret scanning is enabled
2025-03-17T11:46:35Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-03-17T11:46:35Z    INFO    [secret] Please see also https://trivy.dev/v0.60/docs/scanner/secret#recommendation for faster secret detection
2025-03-17T11:46:35Z    INFO    Detected OS     family="debian" version="12.9"
2025-03-17T11:46:35Z    INFO    [debian] Detecting vulnerabilities...   os_version="12" pkg_num=147
2025-03-17T11:46:35Z    INFO    Number of language-specific files       num=1
2025-03-17T11:46:35Z    INFO    [gobinary] Detecting vulnerabilities...
2025-03-17T11:46:35Z    WARN    Using severities from other vendors for some vulnerabilities. Read https://trivy.dev/v0.60/docs/scanner/vulnerability#severity-selection for details.

Report Summary

┌────────────────────────────────────────┬──────────┬─────────────────┬─────────┐
│                 Target                 │   Type   │ Vulnerabilities │ Secrets │
├────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ postgres:13 (debian 12.9)              │  debian  │       152       │    -    │
├────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/bin/gosu                     │ gobinary │       58        │    -    │
├────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ /etc/ssl/private/ssl-cert-snakeoil.key │   text   │        -        │    1    │
└────────────────────────────────────────┴──────────┴─────────────────┴─────────┘
```
------------------------------------

A

```
azureuser@TipTop:~$ trivy image requarks/wiki:latest
2025-03-17T11:58:37Z    INFO    [vuln] Vulnerability scanning is enabled
2025-03-17T11:58:37Z    INFO    [secret] Secret scanning is enabled
2025-03-17T11:58:37Z    INFO    [secret] If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2025-03-17T11:58:37Z    INFO    [secret] Please see also https://trivy.dev/v0.60/docs/scanner/secret#recommendation for faster secret detection
2025-03-17T12:00:16Z    INFO    Detected OS     family="alpine" version="3.21.2"
2025-03-17T12:00:16Z    INFO    [alpine] Detecting vulnerabilities...   os_version="3.21" repository="3.21" pkg_num=71
2025-03-17T12:00:16Z    INFO    Number of language-specific files       num=1
2025-03-17T12:00:16Z    INFO    [node-pkg] Detecting vulnerabilities...
2025-03-17T12:00:16Z    WARN    Using severities from other vendors for some vulnerabilities. Read https://trivy.dev/v0.60/docs/scanner/vulnerability#severity-selection for details.
2025-03-17T12:00:17Z    INFO    Table result includes only package filenames. Use '--format json' option to get the full path to the package file.

Report Summary

┌──────────────────────────────────────────────────────────────────────────────────┬──────────┬─────────────────┬─────────┐
│                                      Target                                      │   Type   │ Vulnerabilities │ Secrets │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ requarks/wiki:latest (alpine 3.21.2)                                             │  alpine  │       28        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ opt/yarn-v1.22.22/package.json                                                   │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/corepack/package.json                                 │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/ansi-reg- │ node-pkg │        0        │    -    │
│ ex/package.json                                                                  │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/emoji-re- │ node-pkg │        0        │    -    │
│ gex/package.json                                                                 │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/string-w- │ node-pkg │        0        │    -    │
│ idth/package.json                                                                │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/node_modules/strip-an- │ node-pkg │        0        │    -    │
│ si/package.json                                                                  │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/cliui/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@isaacs/string-locale-compare/packa- │ node-pkg │        0        │    -    │
│ ge.json                                                                          │          │                 │         │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/agent/package.json           │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/arborist/package.json        │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/config/package.json          │ node-pkg │        0        │    -    │
├──────────────────────────────────────────────────────────────────────────────────┼──────────┼─────────────────┼─────────┤
│ usr/local/lib/node_modules/npm/node_modules/@npmcli/fs/package.json              │ node-pkg │        0        │    -    │
├────────────────────────────────────────────────────
```