Base topology 
The lab deployes an azure VPN gateway into VNET. We deploy a VPN gateway in seperate VNET to similate two different side. 

<img width="432" alt="image" src="https://user-images.githubusercontent.com/61358211/221388545-c588866d-008d-484b-908f-e7cb03193c87.png">



Build Hub Resource Groups, VNETs and Subnets. Note- the Azure VPN GW will take 20+ minutes to provision.

az group create --name Hub --location eastus
az network vnet create --resource-group Hub --name Hub --location eastus --address-prefixes 10.0.0.0/16 --subnet-name HubVM --subnet-prefix 10.0.10.0/24
az network vnet subnet create --address-prefix 10.0.0.0/24 --name GatewaySubnet --resource-group Hub --vnet-name Hub
az network vnet subnet create --address-prefix 10.0.1.0/24 --name Vmsubnet --resource-group Hub --vnet-name Hub
az network public-ip create --name HubVMPubIP --resource-group Hub --location eastus --allocation-method Dynamic
az network nic create --resource-group Hub -n HubVMNIC --location eastus --subnet HubVM --private-ip-address 10.0.10.10 --vnet-name Hub --public-ip-address HubVMPubIP --ip-forwarding true
az vm create -n HubVM -g Hub --image UbuntuLTS --admin-username demouser --admin-password Microsoft123$ --nics HubVMNIC --no-wait 
az network public-ip create --name Azure-VNGpubip --resource-group Hub --allocation-method Dynamic
az network vnet-gateway create --name Azure-VNG --public-ip-address Azure-VNGpubip --resource-group Hub --vnet Hub --gateway-type Vpn --vpn-type RouteBased --sku VpnGw1 --no-wait 




Build On  Another  VNETs and Subnets



az network vnet create --resource-group Hub --name onprem --location eastus --address-prefixes 10.1.0.0/16 --subnet-name HubVM --subnet-prefix 10.0.10.0/24
az network vnet subnet create --address-prefix 10.0.0.0/24 --name GatewaySubnet --resource-group Hub --vnet-name onprem
az network vnet subnet create --address-prefix 10.0.1.0/24 --name Vmsubnet --resource-group Hub --vnet-name onprem

az network public-ip create --name onpremVMPubIP --resource-group Hub --location eastus --allocation-method Dynamic
az network nic create --resource-group Hub -n onpremVMNIC --location eastus --subnet HubVM --private-ip-address 10.1.10.10 --vnet-name onprem --public-ip-address HubVMPubIP --ip-forwarding true
az vm create -n HubVM -g Hub --image UbuntuLTS --admin-username demouser --admin-password Microsoft123$ --nics onpremVMNIC --no-wait 
az network public-ip create --name onprem-VNGpubip --resource-group Hub --allocation-method Dynamic
az network vnet-gateway create --name onprem-VNG --public-ip-address onprem-VNGpubip --resource-group Hub --vnet onprem --gateway-type Vpn --vpn-type RouteBased --sku VpnGw1 --no-wait 



az network public-ip show -g Hub -n Azure-VNGpubip --query "{address: ipAddress}"
az network public-ip show -g Hub -n onprem-VNGpubip --query "{address: ipAddress}"


Do not move forward with the lab until the VPN GW is fully provisioned. The GW is fully provisioned when the previous command returns a public IP adress. Build the local network gaetway. Make sure to change " onprem-VNGpubip" to the correct public IP

az network local-gateway create --gateway-ip-address ***onprem-VNGpubipP*** --name to-onprem --resource-group Hub --local-address-prefixes 10.1.0.0/16
az network vpn-connection create --name to-onprem --resource-group Hub --vnet-gateway1 Azure-VNG -l eastus --shared-key Microsoft --local-gateway2 to-onprem



Create another local-gateway and connection 
az network local-gateway create --gateway-ip-address <Azure-VNGpubip> --name to-azure --resource-group Hub --local-address-prefixes 10.0.0.0/16
az network vpn-connection create --name to-azure --resource-group Hub --vnet-gateway1 onprem-VNG -l eastus --shared-key Microsoft --local-gateway2 to-azure

