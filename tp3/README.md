# TP3B Part1 - Create the base VM

🌞 Créez une VM azure + 🌞 Connexion SSH

```
leobln@leo-vivobook:~/tp-terraform$ az vm create \
  --resource-group rg-tp-terraform-final \
  --name vm-template-master \
  --image almalinux:almalinux-x86_64:10-gen2:latest \
  --size Standard_B1s \
  --location denmarkeast \
  --admin-username azureuser \
  --ssh-key-values "$(cat ~/.ssh/tf.pub)" \
  --public-ip-sku Standard
The default value of '--size' will be changed to 'Standard_D2s_v5' from 'Standard_DS1_v2' in a future release.
Consider upgrading security for your workloads using Azure Trusted Launch VMs. To know more about Trusted Launch, please visit https://aka.ms/TrustedLaunch.
{
  "fqdns": "",
  "id": "/subscriptions/c13f5965-5179-407f-8ed6-53fd295eb4dc/resourceGroups/rg-tp-terraform-final/providers/Microsoft.Compute/virtualMachines/vm-template-master",
  "location": "denmarkeast",
  "macAddress": "7C-ED-8D-6A-C6-F6",
  "powerState": "VM running",
  "privateIpAddress": "10.0.1.5",
  "publicIpAddress": "9.205.153.178",
  "resourceGroup": "rg-tp-terraform-final"
}
leobln@leo-vivobook:~/tp-terraform$ ssh -i ~/.ssh/tf azureuser@9.205.153.178
The authenticity of host '9.205.153.178 (9.205.153.178)' can't be established.
ED25519 key fingerprint is SHA256:F/0XExW+kXIFdpo0vrttb8GhUNsaTvrO335+1SpGafE.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '9.205.153.178' (ED25519) to the list of known hosts.
[azureuser@vm-template-master ~]$ pwd
/home/azureuser

```

🌞 Effectuez la conf suivante :

```
[azureuser@vm-template-master ~]$ sudo dnf install epel-release -y
Last metadata expiration check: 0:09:06 ago on Tue 24 Mar 2026 04:11:40 PM UTC.
Dependencies resolved.
========================================================================================================================================================================================================================================================================================
 Package                                                                           Architecture                                               Version                                                                  Repository                                                  Size
========================================================================================================================================================================================================================================================================================
Installing:
 epel-release                                                                      noarch                                                     10-6.el10                                                                extras                                                      18 k
Installing dependencies:
 selinux-policy-targeted-extra                                                     noarch                                                     42.1.7-1.el10_1.2                                                        crb                                                        711 k
Installing weak dependencies:
 selinux-policy-extra                                                              noarch                                                     42.1.7-1.el10_1.2                                                        crb                                                         34 k

Transaction Summary
========================================================================================================================================================================================================================================================================================
Install  3 Packages

Total download size: 763 k
Installed size: 822 k
Downloading Packages:
(1/3): selinux-policy-extra-42.1.7-1.el10_1.2.noarch.rpm                                                                                                                                                                                                818 kB/s |  34 kB     00:00    
(2/3): epel-release-10-6.el10.noarch.rpm                                                                                                                                                                                                                320 kB/s |  18 kB     00:00    
(3/3): selinux-policy-targeted-extra-42.1.7-1.el10_1.2.noarch.rpm                                                                                                                                                                                       9.1 MB/s | 711 kB     00:00    
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                                                                                                   1.1 MB/s | 763 kB     00:00     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                                                                                                1/1 
  Installing       : selinux-policy-targeted-extra-42.1.7-1.el10_1.2.noarch                                                                                                                                                                                                         1/3 
  Installing       : selinux-policy-extra-42.1.7-1.el10_1.2.noarch                                                                                                                                                                                                                  2/3 
  Installing       : epel-release-10-6.el10.noarch                                                                                                                                                                                                                                  3/3 
  Running scriptlet: epel-release-10-6.el10.noarch                                                                                                                                                                                                                                  3/3 
Many EPEL packages require the CodeReady Builder (CRB) repository.
It is recommended that you run /usr/bin/crb enable to enable the CRB repository.


Installed:
  epel-release-10-6.el10.noarch                                                  selinux-policy-extra-42.1.7-1.el10_1.2.noarch                                                  selinux-policy-targeted-extra-42.1.7-1.el10_1.2.noarch                                                 

Complete!

```

🌞 Go lancer ça :

```
[azureuser@vm-template-master ~]$ # Réinitialisation de cloud-init
sudo cloud-init clean --logs

# Suppression complète de ses datas
sudo rm -rf /var/lib/cloud/*

# On s'assure qu'il est configuré pour se lancer au boot
sudo systemctl enable cloud-init
```

🌞 Proposer une suite de commandes

```
[azureuser@vm-template-master ~]$ # sudo find /var/log -type f -exec truncate -s 0 {} \;
sudo journalctl --vacuum-time=1s
sudo rm -rf /tmp/*
sudo rm -rf /var/tmp/*
history -c && history -w
Vacuuming done, freed 0B of archived journals from /run/log/journal/39f0e281a2834509a1c086ec1ae97a14.
Deleted archived journal /run/log/journal/b81c2c9ebbf748bdba9359785ec10241/system@4936a6748c50433ba7862c0031f8f132-000000000000025c-00064dc68e5e66a5.journal (180K).
Deleted archived journal /run/log/journal/b81c2c9ebbf748bdba9359785ec10241/system@4936a6748c50433ba7862c0031f8f132-00000000000002f7-00064dc68e6cd95e.journal (1000K).
Deleted archived journal /run/log/journal/b81c2c9ebbf748bdba9359785ec10241/system@4936a6748c50433ba7862c0031f8f132-00000000000008e2-00064dc7811f94fd.journal (2.2M).
Vacuuming done, freed 3.3M of archived journals from /run/log/journal/b81c2c9ebbf748bdba9359785ec10241.
Vacuuming done, freed 0B of archived journals from /run/log/journal.
```



```
leobln@leo-vivobook:~/tp-terraform$az vm deallocate --resource-group rg-tp-terraform-final --name vm-template-master
az vm generalize --resource-group rg-tp-terraform-final --name vm-template-master

leobln@leo-vivobook:~/tp-terraform$ az image create \
  --resource-group rg-tp-terraform-final \
  --name alma_chad \
  --source vm-template-master \
  --hyper-v-generation V2
{
  "hyperVGeneration": "V2",
  "id": "/subscriptions/c13f5965-5179-407f-8ed6-53fd295eb4dc/resourceGroups/rg-tp-terraform-final/providers/Microsoft.Compute/images/alma_chad",
  "location": "denmarkeast",
  "name": "alma_chad",
  "provisioningState": "Succeeded",
  "resourceGroup": "rg-tp-terraform-final",
  "sourceVirtualMachine": {
    "id": "/subscriptions/c13f5965-5179-407f-8ed6-53fd295eb4dc/resourceGroups/rg-tp-terraform-final/providers/Microsoft.Compute/virtualMachines/vm-template-master",
    "resourceGroup": "rg-tp-terraform-final"
  },
  "storageProfile": {
    "dataDisks": [],
    "osDisk": {
      "caching": "ReadWrite",
      "diskSizeGB": 30,
      "managedDisk": {
        "id": "/subscriptions/c13f5965-5179-407f-8ed6-53fd295eb4dc/resourceGroups/rg-tp-terraform-final/providers/Microsoft.Compute/disks/vm-template-master_disk1_0ebc715775ef47c89ac6c6ffc3d8f35a",
        "resourceGroup": "rg-tp-terraform-final"
      },
      "osState": "Generalized",
      "osType": "Linux",
      "storageAccountType": "Premium_LRS"
    }
  },
  "tags": {},
  "type": "Microsoft.Compute/images"
}
```

🌞 Lancer une VM à partir de votre template

```

```

🌞 Vérification !

```

```





