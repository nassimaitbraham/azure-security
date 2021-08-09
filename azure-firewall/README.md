<h1> Documentation pour la mise en place d'un firewall azure</h1>

1 - Création d'un ressource group<br/>

nas@Azure:~$ az group create --name AITECH-FW-RG --location eastus<br/>

2 - Mise en place du réseau<br/>

a - Création du réseau virtuel (VNET)<br/>
nas@Azure:~$ az network vnet create \
--name AITECH-FW-VN \<br/>
--resource-group AITECH-FW-RG \
--location eastus \
--address-prefix 10.0.0.0/16<br/>
