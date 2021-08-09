<h1> Documentation pour la mise en place d'un firewall azure</h1>

<h2>1 - Création d'un ressource group</h2>

nas@Azure:~$ az group create --name AITECH-FW-RG --location eastus<br/>

<h2>2 - Mise en place du réseau</h2>

a - Création du réseau virtuel (VNET)<br/>
nas@Azure:~$ az network vnet create \
  --name AITECH-FW-VN \
  --resource-group AITECH-FW-RG \
  --location eastus \
  --address-prefix 10.0.0.0/16<br/><br/>

b - Création subnet du firewall<br/><br/>

nas@Azure:~$ az network vnet subnet create \
   --name AzureFirewallSubnet \
   --resource-group AITECH-FW-RG \
   --vnet-name AITECH-FW-VN   \
   --address-prefix 10.0.1.0/24<br/><br/>
 
 c - Création du subnet pour le serveur de travail<br/>
 
 nas@Azure:~$ az network vnet subnet create \
   --name Workload-SN \
   --resource-group AITECH-FW-RG \
   --vnet-name AITECH-FW-VN   \
   --address-prefix 10.0.2.0/24<br/><br/>
 
 d - Création du subnet pour le serveyr de rebond<br/>

nas@Azure:~$ az network vnet subnet create \
    --name Jump-SN \
   --resource-group AITECH-FW-RG \
   --vnet-name AITECH-FW-VN   \
   --address-prefix 10.0.3.0/24<br/><br/>

<h2>Création des seveurs</h2>

Quand vous y êtes invité, tapez un mot de passe pour la machine virtuelle.

a - Création de la machine virtuelle du serveur de rebond <br/>

nas@Azure:~$ az vm create \
      --resource-group AITECH-FW-RG \
      --name Srv-Jump \
      --location eastus \
      --image win2016datacenter \
      --vnet-name AITECH-FW-VN \
      --subnet Jump-SN \
      --admin-username azureadmin<br/><br/>
     
==> Ouverture du port RDP aux serveur de rebond<br/>

nas@Azure:~$ az vm open-port --port 3389 --resource-group AITECH-FW-RG --name Srv-Jump<br/><br/>

b - Création de la machine virtuelle du serveur de travail<br/><br/>

b.1 - Création d'une carte réseau sans adresse public pour le serveur de travail.<br/><br/>

nas@Azure:~$ az network nic create \
     -g AITECH-FW-RG \
     -n Srv-Work-NIC \
    --vnet-name AITECH-FW-VN \
    --subnet Workload-SN \
    --public-ip-address "" \
    --dns-servers 209.244.0.3 209.244.0.4
b.2 - Création de la machine virtuelle du serveur de travail on attachant la carte réseau du point b.1<br/><br/>

nas@Azure:~$ az vm create \
     --resource-group AITECH-FW-RG \
     --name Srv-Work \
     --location eastus \
     --image win2016datacenter \
     --nics Srv-Work-NIC \
     --admin-username azureadmin



