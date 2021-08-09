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

<h2>3 - Création des seveurs</h2>

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

b.1 - Création d'une carte réseau sans adresse public pour le serveur de travail.<br/>

nas@Azure:~$ az network nic create \
     -g AITECH-FW-RG \
     -n Srv-Work-NIC \
    --vnet-name AITECH-FW-VN \
    --subnet Workload-SN \
    --public-ip-address "" \
    --dns-servers 209.244.0.3 209.244.0.4<br/><br/>
b.2 - Création de la machine virtuelle du serveur de travail on attachant la carte réseau du point b.1<br/>

nas@Azure:~$ az vm create \
     --resource-group AITECH-FW-RG \
     --name Srv-Work \
     --location eastus \
     --image win2016datacenter \
     --nics Srv-Work-NIC \
     --admin-username azureadmin<br/><br/>
     
<h2>4 - Déployement le pare-feu</h2>

a- création du firewall<br/>

nas@Azure:~$ az network firewall create \
    --name AITECH-FW01 \
    --resource-group AITECH-FW-RG \
    --location eastus<br/><br/>

b - création d'une ip public<br/>

nas@Azure:~$ az network public-ip create \
    --name fw-public-ip \
    --resource-group AITECH-FW-RG \
    --location eastus \
    --allocation-method static \
    --sku standard<br/><br/>
	
c - attribution de l'ip pubic au firewall<br/>

nas@Azure:~$ az network firewall ip-config create \
    --firewall-name AITECH-FW01 \
    --name FW-config \
    --public-ip-address fw-public-ip \
    --resource-group AITECH-FW-RG \
    --vnet-name AITECH-FW-VN<br/><br/>
	
d- mise a jour du firewall<br/>

nas@Azure:~$ az network firewall update \
    --name AITECH-FW01 \
    --resource-group AITECH-FW-RG 

c - affichage de l'ip public du firewall<br/><br/>	

nas@Azure:~$ az network public-ip show \
    --name fw-public-ip \
    --resource-group AITECH-FW-RG<br/>

d - affichage de l'ip prive du firewall<br/>
nas@Azure:~$ fwprivaddr="$(az network firewall ip-config list -g AITECH-FW-RG -f AITECH-FW01 --query "[?name=='FW-config'].privateIpAddress" --output tsv)"<br/><br/>

<h2> Créer un itinéraire par défaut </h2>

a - Creation de la table de route<br/>

nas@Azure:~$ az network route-table create \
    --name Firewall-rt-table \
    --resource-group AITECH-FW-RG \
    --location eastus \
    --disable-bgp-route-propagation true<br/><br/>

b - Création de la route (Router toute les addresse IP vers le firewall)<br/>

nas@Azure:~$ az network route-table route create \
  --resource-group AITECH-FW-RG \
  --name DG-Route \
  --route-table-name Firewall-rt-table \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address $fwprivaddr<br/><br/>
  
c - Associez la table de routage au sous-réseau workload-SN<br/>

nas@Azure:~$ az network vnet subnet update \
   -n Workload-SN \
   -g AITECH-FW-RG \
   --vnet-name AITECH-FW-VN \
   --address-prefixes 10.0.2.0/24 \
   --route-table Firewall-rt-table

<h2>5 - Régle d'application dans le firewall</h2>

nas@Azure:~$ az network firewall application-rule create \
   --collection-name App-Coll01 \
   --firewall-name AITECH-FW01 \
   --name Allow-Google \
   --protocols Http=80 Https=443 \
   --resource-group AITECH-FW-RG \
   --target-fqdns www.google.com \
   --source-addresses 10.0.2.0/24 \
   --priority 200 \
   --action Allow<br/><br/> 
 
<h2>6 - Configuration d'une régle de réseau dans le firewall</h2>

nas@Azure:~$ az network firewall network-rule create \
   --collection-name Net-Coll01 \
   --destination-addresses 209.244.0.3 209.244.0.4 \
   --destination-ports 53 \
   --firewall-name AITECH-FW01 \
   --name Allow-DNS \
   --protocols UDP \
   --resource-group AITECH-FW-RG \
   --priority 200 \
   --source-addresses 10.0.2.0/24 \
   --action Allow<br/>
   
<h2>7 - Suppression du ressource group</h2>

nas@Azure:~$ az group delete -n AITECH-FW-RG

