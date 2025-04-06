# 🧑‍🏫 Présentation – Déploiement OpenNebula & Configuration Réseau

---

## 🌐 Partie Réseau – Mise en place de l’environnement

Tout d'abord, on a commencé par **configurer les interfaces réseau des VMs**.

### 🔧 Adaptateurs réseau

Chaque VM est configurée avec :
- Un **adaptateur NAT** pour accéder à Internet
- Un **réseau privé hôte** nommé `TP_Leo` pour la communication interne

---

### 📁 Fichier de configuration de la 3e interface (`enp0s8`)

Sur chaque VM, on a ajouté la config suivante :

```
DEVICE=enp0s8
BOOTPROTO=static
ONBOOT=yes
IPADDR=10.3.1.10
NETMASK=255.255.255.0
```
---

## 📡 Configuration IP & Vérification Réseau

Ensuite, on ajuste les IP selon la machine :
```
| Machine        | IP           |
|----------------|--------------|
| `frontend.one` | `10.3.1.10`  |
| `kvm1.one`     | `10.3.1.11`  |
| `kvm2.one`     | `10.3.1.12`  |
```
---

### ✅ Vérification de connectivité – Ping

#### 🔁 Depuis `kvm1.one` vers `frontend.one`
```
- ping frontend.one
```
**Résultat :**
```
- 4 packets transmitted, 4 received, 0% packet loss  
- rtt min/avg/max = ~0.8 to 1.3 ms
```
---

#### 🔁 Depuis `kvm2.one` vers `frontend.one`
```
- ping frontend.one
```
**Résultat :**
```
- 4 packets transmitted, 4 received, 0% packet loss  
- rtt min/avg/max = ~0.89 to 1.17 ms
```
✅ **Réseau fonctionnel et machines joignables !**

---

## 🛢️ Partie Base de Données – MySQL

L’objectif ici était d’installer **MySQL 8**, requis par OpenNebula.

---

### 📦 Étapes d'installation

#### 1. Ajout du dépôt MySQL
```
- wget https://dev.mysql.com/get/mysql80-community-release-el9-5.noarch.rpm  
- rpm -ivh mysql80-community-release-el9-5.noarch.rpm
```
#### 2. Installation du serveur
```
- dnf install mysql-community-server
```
---

### ⚙️ Démarrage et Configuration

#### Démarrage du service MySQL
```
- systemctl start mysqld  
- systemctl enable mysqld
```
#### Configuration des utilisateurs MySQL
```
- ALTER USER 'root'@'localhost' IDENTIFIED BY 'mangemonSQL1234*';  
- CREATE USER 'oneadmin' IDENTIFIED BY 'mangemondeuxiemeSQL1234*';  
- CREATE DATABASE opennebula;  
- GRANT ALL PRIVILEGES ON opennebula.* TO 'oneadmin';  
- SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```



keyket1234
mysql: mangemonSQL1234*
mangemondeuxiemeSQL1234*
---

## ☁️ Partie OpenNebula – Installation

### 🗂️ Ajout du dépôt OpenNebula

Créer le fichier `/etc/yum.repos.d/opennebula.repo` :

```
[opennebula] name=OpenNebula Community Edition baseurl=https://downloads.opennebula.io/repo/6.10/RedHat/$releasever/$basearch enabled=1 gpgkey=https://downloads.opennebula.io/repo/repo2.key gpgcheck=1 repo_gpgcheck=1
```

Puis mettre en cache les métadonnées :
```
- dnf makecache -y
```
---

### 🧰 Installation OpenNebula
```
- dnf install opennebula opennebula-sunstone opennebula-fireedge
```
---

### 🛠️ Configuration de la base de données

Modifier le fichier `/etc/one/oned.conf` avec les bonnes infos MySQL :
```
DB = [ BACKEND = "mysql", SERVER = "localhost", USER = "oneadmin", PASSWD = "also_here_define_another_strong_password", DB_NAME = "opennebula" ]
```

---

### 👤 Authentification Web UI

Créer le fichier `.one_auth` pour l’utilisateur :
```
- echo "toto:a" > /var/lib/one/.one/one_auth
```
⚠️ Ce fichier doit appartenir à l’utilisateur `oneadmin`.

---

### 🚀 Démarrage des services OpenNebula
```
- systemctl start opennebula opennebula-sunstone  
- systemctl enable opennebula opennebula-sunstone
```
---

## 🔥 Configuration Système – Firewall

Ouverture des ports nécessaires :
```
 firewall-cmd --permanent --add-port=9869/tcp    # WebUI (Sunstone)  
 firewall-cmd --permanent --add-port=22/tcp      # SSH  
 firewall-cmd --permanent --add-port=2633/tcp    # XML-RPC API  
 firewall-cmd --permanent --add-port=4124/tcp    # Monitoring TCP  
 firewall-cmd --permanent --add-port=4124/udp    # Monitoring UDP  
 firewall-cmd --permanent --add-port=29876/tcp   # NoVNC proxy  
 firewall-cmd --reload
```
---

## 🧪 Test Final – Accès WebUI

➡️ Accéder à l'interface web :

[http://10.3.1.11:9869](http://10.3.1.11:9869)

**Identifiants de connexion :**

- **Login** : `toto`  
- **Mot de passe** : `a`

✅ Interface fonctionnelle et accès confirmé !

---
