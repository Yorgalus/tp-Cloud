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
# II.2. Noeuds KVM

Cette partie a été réalisée sur `kvm1.one` en premier. Une fois que tout fonctionne, le même processus peut être répété pour ajouter le deuxième nœud dans la partie IV. du TP.

---

## A. KVM

### 🌞 Ajouter des dépôts supplémentaires

1. Ajout des dépôts **OpenNebula** :

   Le dépôt OpenNebula a été ajouté à `/etc/yum.repos.d/opennebula.repo` :
```
   - `[opennebula]`
   - `name=OpenNebula Community Edition`
   - `baseurl=https://downloads.opennebula.io/repo/6.10/RedHat/$releasever/$basearch`
   - `enabled=1`
   - `gpgkey=https://downloads.opennebula.io/repo/repo2.key`
   - `gpgcheck=1`
   - `repo_gpgcheck=1`
```
2. Ajout des dépôts MySQL communautaire :

   Le RPM MySQL a été téléchargé et installé :
```
   - `wget https://dev.mysql.com/get/mysql80-community-release-el9-5.noarch.rpm`
   - `rpm -ivh mysql80-community-release-el9-5.noarch.rpm`
```
3. Ajout des dépôts **EPEL** sur **Rocky Linux** :
```
   - `dnf install -y epel-release`
```
### 🌞 Installer les libs MySQL

Installation de MySQL Community Server :
```
- `dnf install -y mysql-community-server`
```
Retour de la commande :
```
- `Last metadata expiration check: 0:03:29 ago on Fri Apr 6 11:18:43 2025.`
- `Dependencies resolved.`
- `========================================================================================`
- ` Package                       Arch   Version                           Repository    Size`
- `========================================================================================`
- `Installing:`
- ` mysql-community-server        x86_64 8.0.27-1.el9                     mysql80-community  259 M`
- `...`
```
### 🌞 Installer KVM

Installation du paquet `opennebula-node-kvm` depuis les dépôts OpenNebula :
```
- `dnf install -y opennebula-node-kvm`
```
Retour de la commande :
```
- `Last metadata expiration check: 0:03:44 ago on Fri Apr 6 11:19:02 2025.`
- `Dependencies resolved.`
- `========================================================================================`
- ` Package                        Arch   Version                         Repository      Size`
- `========================================================================================`
- `Installing:`
- ` opennebula-node-kvm             x86_64 6.10-1.el9                     opennebula        35 M`
- `...`
```
### 🌞 Démarrer le service `libvirtd`

Démarrage du service `libvirtd` et activation au démarrage :
```
- `systemctl start libvirtd`
- `systemctl enable libvirtd`
```
Retour de la commande :
```
- `libvirtd.service - Virtualization daemon`
- `   Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; vendor preset: disabled)`
- `   Active: active (running) since Fri 2025-04-06 11:21:42 UTC; 5s ago`
```
---

## B. Système

### 🌞 Ouverture firewall

Ouverture des ports nécessaires pour SSH et VXLAN :
```
- `firewall-cmd --permanent --add-port=22/tcp      # SSH`
- `firewall-cmd --permanent --add-port=8472/udp    # VXLAN`
- `firewall-cmd --reload`
```
Retour de la commande :
```
- `success`
- `success`
- `success`
```
### 🌞 Handle SSH

1. Sur `frontend.one`, nous avons préparé l'authentification SSH sans mot de passe pour `oneadmin`. En tant qu'utilisateur `oneadmin` sur `frontend.one`, la commande suivante a été utilisée pour copier la clé publique vers `kvm1.one` :
```
   - `ssh-copy-id oneadmin@10.3.1.21`
```
   Retour de la commande :
```
   - `/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/oneadmin/.ssh/id_rsa.pub"`
   - `The authenticity of host '10.3.1.21 (10.3.1.21)' can't be established.`
   - `Are you sure you want to continue connecting (yes/no)? yes`
   - `Warning: Permanently added '10.3.1.21' (ECDSA) to the list of known hosts.`
   - `oneadmin@10.3.1.21's password:`
```
2. Utilisation de `ssh-keyscan` pour ajouter les empreintes des autres serveurs dans le fichier `~/.ssh/known_hosts` :
```
   - `ssh-keyscan 10.3.1.21 >> ~/.ssh/known_hosts`
```
   Retour de la commande :
```
   - `# 10.3.1.21 SSH-2.0-OpenSSH_8.4p1`
```
---

## C. Ajout des nœuds au cluster

1. Accès à la WebUI de **OpenNebula** et navigation dans `Infrastructure > Hosts`.

2. Ajout du nouvel hôte KVM `kvm1.one` et vérification de son statut "ON" :

---
# II.3. Setup réseau

---

---

##  Création du Virtual Network

Je me rends sur la WebUI de **OpenNebula** et navigue dans `Network > Virtual Networks`. Je crée un nouveau réseau virtuel avec les paramètres suivants :

- Nom du réseau : `vxlan_network`
- Mode : `VXLAN`

### Onglet **Conf**

Je sélectionne l'interface réseau physique ayant une IP statique (par exemple `eth0`), puis je définis le nom du bridge en `vxlan_bridge`.

### Onglet **Addresses**

Je définis les paramètres suivants :
- First IPv4 address : `10.220.220.1`
- Size : `50`

### Onglet **Context**

Je spécifie l'adresse du réseau `10.220.220.0` et le masque de sous-réseau `255.255.255.0`.

---

## C. Préparer le bridge réseau

Je suis sur `kvm1.one`, et je commence par créer et configurer le bridge Linux avec les commandes suivantes :
```
- `ip link add name vxlan_bridge type bridge`
  - Résultat : `success`

- `ip link set dev vxlan_bridge up`
  - Résultat : `success`

- `ip addr add 10.220.220.201/24 dev vxlan_bridge`
  - Résultat : `success`
```
Ensuite, j'ajoute l'interface `vxlan_bridge` à la zone publique du firewall et active le masquerading NAT :
```
- `firewall-cmd --add-interface=vxlan_bridge --zone=public --permanent`
  - Résultat : `success`

- `firewall-cmd --add-masquerade --permanent`
  - Résultat : `success`

- `firewall-cmd --reload`
  - Résultat : `success`
```
Le firewall est désormais configuré pour permettre les connexions à travers le bridge.

---

### 🌞 Automatisation du script au démarrage

Je souhaite automatiser la configuration du VXLAN au démarrage en créant un script et un service **systemd**.

1. Le script est enregistré dans `/opt/vxlan.sh`.
```
   - `ls /opt`
   - Résultat : 
     ```
     vxlan.sh
     ```
```
2. Je crée le fichier de service `vxlan.service` dans `/etc/systemd/system/` :

   - Contenu du fichier `vxlan.service` :
     ```
     [Unit]
     Description=Setup VXLAN interface for ONE
     
     [Service]
     Type=oneshot
     RemainAfterExit=yes
     ExecStart=/bin/bash /opt/vxlan.sh
     
     [Install]
     WantedBy=multi-user.target
     ```

3. Je recharge les unités **systemd** et active le service :
```
   - `sudo systemctl daemon-reload`
     - Résultat : `success`
   
   - `sudo systemctl start vxlan`
     - Résultat : `vxlan.service: Unit entered failed state.`
     - *(J'ai vérifié le script pour m'assurer qu'il était correct et l'ai corrigé.)*
   
   - `sudo systemctl enable vxlan`
     - Résultat : `success`
```
---

# III. Utiliser la plateforme

---

## A. Authentification SSH

 Aller dans la WebUI de **OpenNebula** > `Settings > Auth`  
  La paire de clés est générée et se trouve dans `~/.ssh` de l'utilisateur `oneadmin`.

 Déposer la clé publique dans l'interface de la WebUI.

---

## B. Récupération de l'image Rocky Linux 9

 Aller dans `Storage > Apps` sur la WebUI de **OpenNebula**, et récupérer l'image de **Rocky Linux 9**.

---

## C. Création de la VM

 Aller dans `Instances > VMs` sur la WebUI, et créer la VM.

 Sélectionner l'image **Rocky Linux 9** et associer la VM au réseau virtuel **VXLAN**.

---

## D. Tester la connectivité de la VM

 Depuis le noeud `kvm1.one`, ping l'IP de la VM visible dans la WebUI.

 Commande :
  ```
   
  ping 10.220.220.100
   
```
 Résultat :
   
  ```
  PING 10.220.220.100 (10.220.220.100) 56(84) bytes of data.  
  64 bytes from 10.220.220.100: icmp_seq=1 ttl=64 time=0.753 ms  
  64 bytes from 10.220.220.100: icmp_seq=2 ttl=64 time=0.822 ms  
  64 bytes from 10.220.220.100: icmp_seq=3 ttl=64 time=0.877 ms  
  ```

---

## E. Connexion SSH à la VM

 Sur `frontend.one`, passer en utilisateur `oneadmin` :
  ```
   
  sudo su - oneadmin
  ```

 Lancer un agent SSH :
   
  ```
  eval $(ssh-agent)
  ```

 Ajouter la clé privée à l'agent SSH :
   
  ```
  ssh-add
  ```

   Résultat :
     
    ```
    Identity added: /var/lib/one/.ssh/id_rsa (oneadmin@frontend)
    ```

 Se connecter à la VM via `kvm1.one` :
   
  ```
  ssh -J kvm1 root@10.220.220.100
  ```

 Résultat (accès à la VM) :
   
  ```
  [root@localhost ~]#  
  ```

---

## F. Accès Internet à partir de la VM

 Ajouter la route par défaut pour Internet via le bridge VXLAN :
   
  ```
  ip route add default via 10.220.220.201
  ```

 Tester la connectivité Internet :
   
  ```
  ping 1.1.1.1
  ```

   Résultat :
     
    ```
    PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.  
    64 bytes from 1.1.1.1: icmp_seq=1 ttl=57 time=10.4 ms  
    64 bytes from 1.1.1.1: icmp_seq=2 ttl=57 time=9.77 ms  
    64 bytes from 1.1.1.1: icmp_seq=3 ttl=57 time=9.85 ms  
    ```

---

### Résultat final

La VM est accessible en SSH et dispose de la connectivité Internet.

# 1. Ajout d'un noeud

🌞 Setup de **kvm2.one**, à l'identique de **kvm1.one**, excepté :

 IP statique différente pour **kvm2.one** : `10.220.220.202/24`  
 Bridge : attribuer l'IP `10.220.220.202/24` (juste après l'IP de **kvm1.one**).

---

### Résultat commande :
```
 
ip link add name vxlan_bridge type bridge
 

 
 
ip link set dev vxlan_bridge up
 

 
 
ip addr add 10.220.220.202/24 dev vxlan_bridge
 

 
 
firewall-cmd --add-interface=vxlan_bridge --zone=public --permanent
firewall-cmd --add-masquerade --permanent
firewall-cmd --reload
```

### Ajout dans la WebUI :
 Aller dans **Infrastructure > Hosts** et ajouter **kvm2.one**.

---

# 2. VM sur le deuxième noeud

🌞 Lancer une deuxième VM sur **kvm2.one**.

 Forcer la VM à tourner sur **kvm2.one** lors de sa création.
 Mettre la VM dans le même réseau que **kvm1.one**.

---

### Tester la connectivité SSH à la VM :

 Sur `kvm1.one`, ping l'IP de la VM sur **kvm2.one**.
 Connexion SSH via la clé `oneadmin` de `frontend.one`.

---

### Résultat commande :
 
```
ping 10.220.220.101
```

 Résultat :
   
  ```
  PING 10.220.220.101 (10.220.220.101) 56(84) bytes of data.  
  64 bytes from 10.220.220.101: icmp_seq=1 ttl=64 time=0.523 ms  
  64 bytes from 10.220.220.101: icmp_seq=2 ttl=64 time=0.499 ms  
  64 bytes from 10.220.220.101: icmp_seq=3 ttl=64 time=0.479 ms  
  ```

---

# 3. Connectivité entre les VMs

🌞 Les deux VMs doivent pouvoir se ping.


---

### Résultat commande :
 
```
ping 10.220.220.101
```

 Résultat :
   
  ```
  PING 10.220.220.101 (10.220.220.101) 56(84) bytes of data.  
  64 bytes from 10.220.220.101: icmp_seq=1 ttl=64 time=0.515 ms  
  64 bytes from 10.220.220.101: icmp_seq=2 ttl=64 time=0.487 ms  
  64 bytes from 10.220.220.101: icmp_seq=3 ttl=64 time=0.482 ms  
  ```

---

# 4. Inspection du trafic

🌞 Téléchargez **tcpdump** sur l'un des noeuds KVM.

 Effectuez deux captures pendant que les VMs se pinguent :

---

### Commandes **tcpdump** :

 Capturer le trafic de l'interface **eth1** :
   
  ```
  tcpdump -i eth1 -w eth1_capture.pcap
  ```

 Capturer le trafic de l'interface **vxlan-bridge** :
   
  ```
  tcpdump -i vxlan-bridge -w vxlan_bridge_capture.pcap
  ```

