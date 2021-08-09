<h1> Documentation pour la mise en place d'un firewall azure</h1>

1 - Création d'un ressource group<br/>

nas@Azure:~$ az group create --name AITECH-FW-RG --location eastus<br/>

2 - Mise en place du réseau<br/>

a - Création du réseau virtuel (VNET)<br/><br/>
nas@Azure:~$ az network vnet create \
  --name AITECH-FW-VN \<br/>
  --resource-group AITECH-FW-RG \
  --location eastus \
  --address-prefix 10.0.0.0/16<br/><br/>

b - Création subnet du firewall<br/><br/>

nas@Azure:~$ az network vnet subnet create \
   --name AzureFirewallSubnet \
   --resource-group AITECH-FW-RG \
   --vnet-name AITECH-FW-VN   \
   --address-prefix 10.0.1.0/24<br/><br/>
 
 C - Création du subnet pour le serveur de travail<br/><br/>
 
 nas@Azure:~$ az network vnet subnet create \
   --name Workload-SN \
   --resource-group AITECH-FW-RG \
   --vnet-name AITECH-FW-VN   \
   --address-prefix 10.0.2.0/24<br/><br/>
 
 D - Création du subnet pour le serveyr de rebond<br/><br/>

nas@Azure:~$ az network vnet subnet create \
    --name Jump-SN \
   --resource-group AITECH-FW-RG \
   --vnet-name AITECH-FW-VN   \
   --address-prefix 10.0.3.0/24<br/><br/>
