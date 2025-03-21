#  Partie I 

##  Partie I : Création et Gestion d'une VM Azure

###  Créez une VM depuis le Azure CLI
````
az vm create -g TP_group -n super_vm --image Ubuntu2204 --admin-username azureuser --ssh-key-values home/keyket/.ssh/id_rsa.pub
````

###  Vérifier les services `walinuxagent` et `cloud-init`

**Vérification du service `walinuxagent`**

````
azureuser@supervm:~$ systemctl status walinuxagent
● walinuxagent.service - Azure Linux Agent
     Loaded: loaded (/lib/systemd/system/walinuxagen>
     Active: active (running) since Tue 2025-03-18 1>
   Main PID: 601 (python3)
      Tasks: 6 (limit: 4023)
     Memory: 37.4M
        CPU: 1.135s
     CGroup: /system.slice/walinuxagent.service
             ├─601 /usr/bin/python3 -u /usr/sbin/waa>
lines 1-9...skipping...
● walinuxagent.service - Azure Linux Agent
     Loaded: loaded (/lib/systemd/system/walinuxagent.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2025-03-18 13:23:19 UTC; 39s ago
   Main PID: 601 (python3)
      Tasks: 6 (limit: 4023)
     Memory: 37.4M
        CPU: 1.135s
     CGroup: /system.slice/walinuxagent.service
             ├─601 /usr/bin/python3 -u /usr/sbin/waagent -daemon
             └─696 python3 -u bin/WALinuxAgent-2.12.0.2-py3.9.egg -run-exthandlers

Mar 18 13:23:23 supervm python3[696]: Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
Mar 18 13:23:23 supervm python3[696]:     pkts      bytes target     prot opt in     out     source               destination
Mar 18 13:23:23 supervm python3[696]:        0        0 ACCEPT     tcp  --  *      *       0.0.0.0/0            168.63.129.16       >Mar 18 13:23:23 supervm python3[696]:       76     9682 ACCEPT     tcp  --  *      *       0.0.0.0/0            168.63.129.16       >Mar 18 13:23:23 supervm python3[696]:        0        0 DROP       tcp  --  *      *       0.0.0.0/0            168.63.129.16       >Mar 18 13:23:23 supervm python3[696]: 2025-03-18T13:23:23.757292Z INFO EnvHandler ExtHandler Set block dev timeout: sdb with timeout>Mar 18 13:23:23 supervm python3[696]: 2025-03-18T13:23:23.757430Z INFO EnvHandler ExtHandler Set block dev timeout: sda with timeout>Mar 18 13:23:23 supervm python3[696]: 2025-03-18T13:23:23.760188Z INFO ExtHandler ExtHandler ProcessExtensionsGoalState completed [i>Mar 18 13:23:23 supervm python3[696]: 2025-03-18T13:23:23.793138Z INFO ExtHandler ExtHandler Looking for existing remote access user>Mar 18 13:23:23 supervm python3[696]: 2025-03-18T13:23:23.818689Z INFO ExtHandler ExtHandler [HEARTBEAT] Agent WALinuxAgent-2.12.0.2>
````
----

####  Vérification du service `cloud-init`

````
azureuser@supervm:~$ systemctl status cloud-init
● cloud-init.service - Cloud-init: Network Stage
     Loaded: loaded (/lib/systemd/system/cloud-init.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2025-03-18 13:23:18 UTC; 9min ago
    Process: 467 ExecStart=/usr/bin/cloud-init init (code=exited, status=0/SUCCESS)
   Main PID: 467 (code=exited, status=0/SUCCESS)
        CPU: 605ms

Mar 18 13:23:18 supervm cloud-init[471]: ci-info: |   3   |    local    |    ::   |    eth0   |   U   |
Mar 18 13:23:18 supervm cloud-init[471]: ci-info: |   4   |  multicast  |    ::   |    eth0   |   U   |
Mar 18 13:23:18 supervm cloud-init[471]: ci-info: +-------+-------------+---------+-----------+-------+
Mar 18 13:23:18 supervm ntfs-3g[482]: Version 2021.8.22 integrated FUSE 28
Mar 18 13:23:18 supervm ntfs-3g[482]: Mounted /dev/sdb1 (Read-Only, label "Temporary Storage", NTFS 3.1)
Mar 18 13:23:18 supervm ntfs-3g[482]: Cmdline options: ro
Mar 18 13:23:18 supervm ntfs-3g[482]: Mount options: ro,allow_other,nonempty,relatime,fsname=/dev/sdb1,blkdev,blksize=4096
Mar 18 13:23:18 supervm ntfs-3g[482]: Ownership and permissions disabled, configuration type 7
Mar 18 13:23:18 supervm ntfs-3g[482]: Unmounting /dev/sdb1 (Temporary Storage)
Mar 18 13:23:18 supervm systemd[1]: Finished Cloud-init: Network Stage.
````

-----

````
azureuser@supervm:~$ cloud-init status
status: done
````

###  Supprimer les ressources après utilisation

````
az vm delete --resource-group TP_group --name super_vm --yes
````
-----

##  Partie II : Un petit LAN

###  Créez deux VMs depuis le Azure CLI

####  Créer un réseau virtuel et un sous-réseau

````
az network vnet create --resource-group TP_group --name LAN --subnet-name subnet
{
  "newVNet": {
    "addressSpace": {
      "addressPrefixes": [
        "10.0.0.0/16"
      ]
    },
    "enableDdosProtection": false,
    "etag": "W/\"0b0efb3f-1699-48ba-ac29-8b26542433d4\"",
    "id": "/subscriptions/cb120904-8a88-4a15-b2d7-66c74fade683/resourceGroups/TP_group/providers/Microsoft.Network/virtualNetworks/LAN",
    "location": "southafricanorth",
    "name": "LAN",
    "privateEndpointVNetPolicies": "Disabled",
    "provisioningState": "Succeeded",
    "resourceGroup": "TP_group",
    "resourceGuid": "a72cc73d-9568-4005-bb4c-66ddd867bf9a",
    "subnets": [
      {
        "addressPrefix": "10.0.0.0/24",
        "delegations": [],
        "etag": "W/\"0b0efb3f-1699-48ba-ac29-8b26542433d4\"",
        "id": "/subscriptions/cb120904-8a88-4a15-b2d7-66c74fade683/resourceGroups/TP_group/providers/Microsoft.Network/virtualNetworks/LAN/subnets/subnet",
        "name": "subnet",
        "privateEndpointNetworkPolicies": "Disabled",
        "privateLinkServiceNetworkPolicies": "Enabled",
        "provisioningState": "Succeeded",
        "resourceGroup": "TP_group",
        "type": "Microsoft.Network/virtualNetworks/subnets"
      }
    ],
    "type": "Microsoft.Network/virtualNetworks",
    "virtualNetworkPeerings": []
  }
}
````
----

####  Création des deux VMs

**VM1**

````
az vm create -g TP_group -n VM1 --image Ubuntu2204 --admin-username azureuser --ssh-key-values home/keyket/.ssh/id_rsa.pub --vnet-name LAN --subnet subnet
{
  "fqdns": "",
  "id": "/subscriptions/cb120904-8a88-4a15-b2d7-66c74fade683/resourceGroups/TP_group/providers/Microsoft.Compute/virtualMachines/VM1",
  "location": "southafricanorth",
  "macAddress": "00-22-48-65-63-86",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "4.221.171.231",
  "resourceGroup": "TP_group",
  "zones": ""
}
````
----

**VM2**

````
az vm create -g TP_group -n VM2 --image Ubuntu2204 --admin-username azureuser --ssh-key-values C:\Users\keyket\.ssh\id_rsa.pub --vnet-name LAN --subnet subnet
{
  "fqdns": "",
  "id": "/subscriptions/cb120904-8a88-4a15-b2d7-66c74fade683/resourceGroups/TP_group/providers/Microsoft.Compute/virtualMachines/VM2",
  "location": "southafricanorth",
  "macAddress": "00-22-48-65-AC-53",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "98.70.74.27",
  "resourceGroup": "TP_group",
  "zones": ""
}
````

### Vérification des adresses IP privées

**VM1**

````

azureuser@VM1:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:22:48:65:63:86 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.4/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::222:48ff:fe65:6386/64 scope link
       valid_lft forever preferred_lft forever
````

----

**VM2**

````

azureuser@VM2:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:22:48:65:ac:53 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.5/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::222:48ff:fe65:ac53/64 scope link
       valid_lft forever preferred_lft forever
````

-----

###  Test de connectivité (Ping)

````
azureuser@VM1:~$ ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=2.62 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=7.68 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=1.55 ms
64 bytes from 10.0.0.5: icmp_seq=4 ttl=64 time=1.70 ms
64 bytes from 10.0.0.5: icmp_seq=5 ttl=64 time=1.65 ms
64 bytes from 10.0.0.5: icmp_seq=6 ttl=64 time=1.69 ms
^C
--- 10.0.0.5 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5009ms
rtt min/avg/max/mdev = 1.545/2.813/7.676/2.204 ms
````

-----

**Ping `VM2` to `VM1`**
````
azureuser@VM2:~$ ping 10.0.0.4
PING 10.0.0.4 (10.0.0.4) 56(84) bytes of data.
64 bytes from 10.0.0.4: icmp_seq=1 ttl=64 time=1.56 ms
64 bytes from 10.0.0.4: icmp_seq=2 ttl=64 time=1.49 ms
64 bytes from 10.0.0.4: icmp_seq=3 ttl=64 time=1.63 ms
64 bytes from 10.0.0.4: icmp_seq=4 ttl=64 time=1.60 ms
64 bytes from 10.0.0.4: icmp_seq=5 ttl=64 time=1.78 ms
64 bytes from 10.0.0.4: icmp_seq=6 ttl=64 time=1.55 ms
^C
--- 10.0.0.4 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5009ms
rtt min/avg/max/mdev = 1.488/1.600/1.776/0.090 ms
````

-----

###  Supprimer les ressources après utilisation
````
az vm delete -g TP_group -n VM1 --yes

az vm delete -g TP_group -n VM2 --yes
````

**Suppremier les interfaces réseaux**
````
az network nic list --resource-group TP_group --output table
AuxiliaryMode    AuxiliarySku    DisableTcpStateTracking    EnableAcceleratedNetworking    EnableIPForwarding    Location          MacAddress         Name      NicType    Primary    ProvisioningState    ResourceGroup    ResourceGuid                          VnetEncryptionSupported
---------------  --------------  -------------------------  -----------------------------  --------------------  ----------------  -----------------  --------  ---------  ---------  -------------------  ---------------  ------------------------------------  -------------------------
None             None            False                      False                          False                 southafricanorth  60-45-BD-41-FB-A8  tp632     Standard   True       Succeeded            TP_group         19e45df2-ba98-4d8c-9185-42ceac76d223  False
None             None            False                      False                          False                 southafricanorth  00-22-48-65-63-86  VM1VMNic  Standard              Succeeded            TP_group         5a1576e5-640a-4ea4-87bc-702afe23011b  False
None             None            False                      False                          False                 southafricanorth  00-22-48-65-AC-53  VM2VMNic  Standard              Succeeded            TP_group         ac671ce7-bf70-4d82-840d-3e2970006092  False
````

---
````
az network nic delete --resource-group TP_group --name VM1VMNIC

az network nic delete --resource-group TP_group --name VM2VMNIC
````

##  Partie III : Utilisation de Terraform avec Azure

###  Préparation

````bash
mkdir terraform-azure-tp
cd terraform-azure-tp
````

### Création du fichier `main.tf`
````powershell
New-Item -Path . -Name "main.tf" -ItemType "File"
````

####  Contenu du fichier `main.tf`

```cat .\main.tf
provider "azurerm" {
  features {}
  subscription_id = " "
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}
resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}
resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}
resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}
resource "azurerm_network_interface" "main" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}
resource "azurerm_network_interface" "internal" {
  name                = "${var.prefix}-nic2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}
resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = azurerm_network_interface.main.private_ip_address
  }
}
resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}
resource "azurerm_linux_virtual_machine" "main" {
  name                            = "${var.prefix}-vm"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_F2"
  admin_username                  = "azureuser"
  network_interface_ids = [
    azurerm_network_interface.main.id,
    azurerm_network_interface.internal.id,
  ]
  admin_ssh_key {
    username   = "azureuser"
    public_key = file("home/keyket/.ssh/id_rsa.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}
```

###  Création du fichier `variables.tf`

````powershell
New-Item -Path . -Name "variables.tf" -ItemType "File"
````

####  Contenu du fichier `variables.tf`

```terraform
variable "prefix" {
  description = "Préfixe pour nommer les ressources"
  default     = "tp3"
}

variable "location" {
  description = "Emplacement de déploiement"
  default     = "West Europe"
}
```

---

````
terraform init

terraform plan

terraform apply
````

----
##  Déploiement Terraform et Vérifications Azure

### 1. Vérification du déploiement
````
 az vm list

 az vm show --name tp2magueule-vm --resource-group tp2magueule-resources

 az group list
````

### 2. Connexion SSH à `node1`

````
ssh azureuser@20.204.159.40

azureuser@tp2magueule-node1:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0d:3a:20:c3:f2 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.5/24 metric 100 brd 10.0.2.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::20d:3aff:fe20:c3f2/64 scope link
       valid_lft forever preferred_lft forever
````
 
-----

### 3. Connexion SSH via JumpHost (`node1` → `node2`)
````
ssh -J azureuser@20.204.159.40 azureuser@10.0.2.4
````

### 4. Test de Ping depuis `node1` vers `node2`
````
azureuser@tp2magueule-node1:~$ ping 10.0.2.4
PING 10.0.2.4 (10.0.2.4) 56(84) bytes of data.
64 bytes from 10.0.2.4: icmp_seq=1 ttl=64 time=1.14 ms
64 bytes from 10.0.2.4: icmp_seq=2 ttl=64 time=0.991 ms
64 bytes from 10.0.2.4: icmp_seq=3 ttl=64 time=1.09 ms
64 bytes from 10.0.2.4: icmp_seq=4 ttl=64 time=1.38 ms
^C
--- 10.0.2.4 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 0.991/1.149/1.375/0.141 ms
````

----

### 4. Fichier de Configuration `cloud-init.txt`

````
#cloud-config
users:
  - name: azureuser
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: docker
    shell: /bin/bash
    passwd:
    ssh_authorized_keys:
      - 

apt:
  sources:
    docker.list:
      source: deb [arch=amd64] https://download.docker.com/linux/ubuntu $RELEASE stable
      keyid: 9DC858229FC7DD38854AE2D88D81803C0EBFCD88

packages:
  - docker-ce
  - docker-ce-cli

runcmd:
  - docker pull alpine:latest
````

### 4. Configuration Terraform (`main.tf`)
````
provider "azurerm" {
  features {}
  subscription_id = ""
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_public_ip" "vm_pip" {
  name                = "${var.prefix}-vm-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "vm_nic" {
  name                = "${var.prefix}-vm-nic"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "public"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.vm_pip.id
  }
}

resource "azurerm_network_security_group" "ssh" {
  name                = "${var.prefix}-ssh-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "allow_ssh"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "vm_assoc" {
  network_interface_id      = azurerm_network_interface.vm_nic.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                  = "${var.prefix}-vm"
  resource_group_name   = azurerm_resource_group.main.name
  location              = azurerm_resource_group.main.location
  size                  = "Standard_F2"
  admin_username        = "azureuser"
  network_interface_ids = [azurerm_network_interface.vm_nic.id]

  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  custom_data = filebase64("${path.module}/cloud-init.txt")

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}
````

### 4. Connexion à la VM Azure
````
ssh azureuser@104.45.24.67
````

### 5. Vérification de l'installation de Docker
````
azureuser@tp2magueule-vm:~$ docker -v
Docker version 28.0.2, build 0442a73
````

###  **Configurations additionnelles Terraform & cloud-init**
````
provider "azurerm" {
  features {}
  subscription_id = " "
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_public_ip" "vm_pip" {
  name                = "${var.prefix}-vm-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "vm_nic" {
  name                = "${var.prefix}-vm-nic"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "public"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.vm_pip.id
  }
}

resource "azurerm_network_security_group" "vm_nsg" {
  name                = "${var.prefix}-vm-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "allow_ssh"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "allow_wikijs"
    priority                   = 200
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "10101"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "vm_assoc" {
  network_interface_id      = azurerm_network_interface.vm_nic.id
  network_security_group_id = azurerm_network_security_group.vm_nsg.id
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                  = "${var.prefix}-vm"
  resource_group_name   = azurerm_resource_group.main.name
  location              = azurerm_resource_group.main.location
  size                  = "Standard_F2"
  admin_username        = "azureuser"
  network_interface_ids = [azurerm_network_interface.vm_nic.id]

  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  custom_data = filebase64("${path.module}/cloud-init.txt")

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}
````

----

### 6. Vérification HTTP (WikiJS)
````
curl 104.45.24.67:10101


StatusCode        : 200
StatusDescription : OK
Content           : <!DOCTYPE html><html><head><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta charset="UTF-8"><meta
                    name="viewport" content="user-scalable=yes, width=device-width, initial-scale=1, maximum-sca...
RawContent        : HTTP/1.1 200 OK
                    Vary: Accept-Encoding
                    Connection: keep-alive
                    Keep-Alive: timeout=5
                    Content-Length: 1315
                    Content-Type: text/html; charset=utf-8
                    Date: Thu, 20 Mar 2025 11:40:09 GMT
                    ETag: W/"523-R...
Forms             : {}
Headers           : {[Vary, Accept-Encoding], [Connection, keep-alive], [Keep-Alive, timeout=5], [Content-Length, 1315]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 1315
````