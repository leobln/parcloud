# 1. Starting blocks

🌞 Déterminer quel algorithme de chiffrement utiliser pour vos clés

```
Comme algorithme de chiffrement on va utiliser EdEd25519
Si tu veux une source fiable voici la mienne je suis aller sur Mozila infosec
```

🌞 Générer une paire de clés pour ce TP

```
ssh-keygen -t ed25519 -f ~/.ssh/cloud_tp1 -C "tp_cloud_user"
```

🌞 Configurer un agent SSH sur votre poste

```
nano ~/.ssh/config

Host azure-tp
    HostName <IP_de_ma_vm>
    User azureuser
    IdentityFile ~/.ssh/cloud_tp1
    IdentitiesOnly yes
```

# II. Spawn des VMs

🌞 Connectez-vous en SSH à la VM pour preuve

```
leobln@leo-vivobook:~$ ssh -i ~/.ssh/cloud_tp1 azureuser@68.221.161.42
The authenticity of host '68.221.161.42 (68.221.161.42)' can't be established.
ED25519 key fingerprint is SHA256:VUwmUJyUWbNiZjrohvNrTMiCO0+iRl1H8Yvpos025no.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '68.221.161.42' (ED25519) to the list of known hosts.
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.17.0-1008-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Mar 24 09:02:32 UTC 2026

  System load:  0.14              Processes:             123
  Usage of /:   5.7% of 28.02GB   Users logged in:       0
  Memory usage: 3%                IPv4 address for eth0: 10.0.0.4
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

azureuser@m-tp2:~$ ls
azureuser@m-tp2:~$ pwd
/home/azureuser
```

🌞 Créez une VM depuis le Azure CLI

```
leobln@leo-vivobook:~$ az group create --name rg-tp-cloud-cli --location denmarkeast
{
  "id": "/subscriptions/c13f5965-5179-407f-8ed6-53fd295eb4dc/resourceGroups/rg-tp-cloud-cli",
  "location": "denmarkeast",
  "managedBy": null,
  "name": "rg-tp-cloud-cli",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
leobln@leo-vivobook:~$ az vm create \
  --resource-group rg-tp-cloud-cli \
  --name vm-alma-tp2 \
  --image almalinux:almalinux-x86_64:10-gen2:latest \
  --size Standard_B1s \
  --admin-username azureuser \
  --ssh-key-values "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFmQUdD35v+ZXIxt4Ml/FcHGIbF7erQFlw2OOllgNsLN tp_cloud_user" \
  --location denmarkeast
The default value of '--size' will be changed to 'Standard_D2s_v5' from 'Standard_DS1_v2' in a future release.
Consider upgrading security for your workloads using Azure Trusted Launch VMs. To know more about Trusted Launch, please visit https://aka.ms/TrustedLaunch.
{
  "fqdns": "",
  "id": "/subscriptions/c13f5965-5179-407f-8ed6-53fd295eb4dc/resourceGroups/rg-tp-cloud-cli/providers/Microsoft.Compute/virtualMachines/vm-alma-tp2",
  "location": "denmarkeast",
  "macAddress": "7C-ED-8D-37-E9-30",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "9.205.156.115",
  "resourceGroup": "rg-tp-cloud-cli"
}
```

🌞 Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique

```
leobln@leo-vivobook:~$ ssh azureuser@9.205.156.115
The authenticity of host '9.205.156.115 (9.205.156.115)' can't be established.
ED25519 key fingerprint is SHA256:4uSIicSWk2KnrIFcD2PMW/QWdPJGSH4wnEn9eZJ2Oyk.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '9.205.156.115' (ED25519) to the list of known hosts.

1 device has a firmware upgrade available.
Run `fwupdmgr get-upgrades` for more information.

[azureuser@vm-alma-tp2 ~]$ ls
[azureuser@vm-alma-tp2 ~]$ pwd
/home/azureuser
```

🌞 Une fois connecté, prouvez la présence... waagent.service/cloud-init.service

```
[azureuser@vm-alma-tp2 ~]$ systemctl status waagent.service
● waagent.service - Azure Linux Agent
     Loaded: loaded (/usr/lib/systemd/system/waagent.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-03-24 10:11:19 UTC; 2min 47s ago


[azureuser@vm-alma-tp2 ~]$ cloud-init status
status: done
[azureuser@vm-alma-tp2 ~]$ systemctl status cloud-init.service
● cloud-init.service - Cloud-init: Network Stage
     Loaded: loaded (/usr/lib/systemd/system/cloud-init.service; enabled; preset: enabled)
     Active: active (exited) since Tue 2026-03-24 10:11:19 UTC; 3min 37s ago

```

📁 Fichiers à rendre

main.tf

```
provider "azurerm" {
  features {}
  subscription_id            = var.subscription_id
  skip_provider_registration = true
}

resource "azurerm_resource_group" "main" {
  name     = var.resource_group_name
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "vm-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "main" {
  name                 = "vm-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "main" {
  name                = "vm-ip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_network_interface" "main" {
  name                = "vm-nic"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.main.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.main.id
  }
}

resource "azurerm_linux_virtual_machine" "main" {
  name                = "super-vm"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B1s"
  admin_username      = var.admin_username

  # Sécurité pour accepter Ed25519
  secure_boot_enabled = false
  vtpm_enabled        = false

  network_interface_ids = [
    azurerm_network_interface.main.id,
  ]

  admin_ssh_key {
    username   = var.admin_username
    public_key = trimspace(file(var.public_key_path))
  }

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
    name                 = "vm-os-disk"
  }

  source_image_reference {
    publisher = "almalinux"
    offer     = "almalinux-x86_64"
    sku       = "10-gen2"
    version   = "latest"
  }
}

```

variables.tf

```
 variables.tf

variable "resource_group_name" {
  type        = string
  description = "Name of the resource group"
}

variable "location" {
  type        = string
  default     = "East US"
  description = "Azure region"
}

variable "admin_username" {
  type        = string
  description = "Admin username for the VM"
}

variable "public_key_path" {
  type        = string
  description = "Path to your SSH public key"
}

variable "subscription_id" {
  type        = string
  description = "Azure subscription ID"
}

```

terraform.tfvars

```
esource_group_name = "rg-tp-terraform-final"
admin_username      = "azureuser"
public_key_path     = "~/.ssh/tf.pub"
location            = "denmarkeast"
subscription_id     = "c13f5965-5179-407f-8ed6-53fd295eb4dc"
```

🌞 Prouvez avec une connexion SSH sur l'IP publique que la VM est up

```
leobln@leo-vivobook:~/tp-terraform$ ssh -i ~/.ssh/tf azureuser@9.205.153.113
The authenticity of host '9.205.153.113 (9.205.153.113)' can't be established.
ED25519 key fingerprint is SHA256:yr0RqSSi9acszWMGT/RCL3PYrY+Kqw8NEuCPChtk2+s.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '9.205.153.113' (ED25519) to the list of known hosts.

1 device has a firmware upgrade available.
Run `fwupdmgr get-upgrades` for more information.

[azureuser@super-vm ~]$ pwd
/home/azureuser
```

