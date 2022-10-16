# Application gateway ingress controller add-on for an existing AKS cluster with an existing application gateway

$RESOURCE_GROUP=aks-rg
$LOCATION=westus
$AKS_ClUSTERNAME=myAKSCluster

az group create -n $RESOURCE_GROUP -l $LOCATION

az aks create -n $AKS_ClUSTERNAME -g $RESOURCE_GROUP -l $LOCATION --network-plugin azure --enable-managed-identity

# Creating App Gateway and public IP

az network public-ip create -n myPublicIp -g $RESOURCE_GROUP --allocation-method Static --sku Standard

az network vnet create -n myVnet -g $RESOURCE_GROUP --address-prefix 10.0.0.0/16 --subnet-name mySubnetA --subnet-prefix 10.0.0.0/24

az network application-gateway create -n myApplicationGateway -l $LOCATION -g $RESOURCE_GROUP --sku Standard_v2 --public-ip-address myPublicIp --vnet-name myVnet --subnet mySubnetA --priority 100

# Enabling AGIC add-on in AKS

$appgwId=az network application-gateway show -n myApplicationGateway -g $RESOURCE_GROUP -o tsv --query "id"

az aks enable-addons -n $AKS_ClUSTERNAME -g $RESOURCE_GROUP -a ingress-appgw --appgw-id $appgwId

# Peer the two virtual networks together

Since you deployed the AKS cluster in its own virtual network and the Application gateway in another virtual network, you'll need to peer the two virtual networks together in order for traffic to flow from the Application gateway to the pods in the cluster. Peering the two virtual networks requires running the Azure CLI command two separate times, to ensure that the connection is bi-directional. The first command will create a peering connection from the Application gateway virtual network to the AKS virtual network; the second command will create a peering connection in the other direction.

$nodeResourceGroup=az aks show -n $AKS_ClUSTERNAME -g $RESOURCE_GROUP -o tsv --query "nodeResourceGroup"

$aksVnetName=az network vnet list -g $nodeResourceGroup -o tsv --query "[0].name"

$aksVnetId=az network vnet show -n $aksVnetName -g $nodeResourceGroup -o tsv --query "id"

az network vnet peering create -n AppGWtoAKSVnetPeering -g $RESOURCE_GROUP --vnet-name myVnet --remote-vnet $aksVnetId --allow-vnet-access

$appGWVnetId=az network vnet show -n myVnet -g $RESOURCE_GROUP -o tsv --query "id"

az network vnet peering create -n AKStoAppGWVnetPeering -g $nodeResourceGroup --vnet-name $aksVnetName --remote-vnet $appGWVnetId --allow-vnet-access


# Sample Application

```kubectl apply -f https://raw.githubusercontent.com/Azure/application-gateway-kubernetes-ingress/master/docs/examples/aspnetapp.yaml```

# Create a DNS Zone for custom Domain

**Using Azure CLI***
az network dns zone create -g aksdemocluster-rg -n rajdevopslearn.online

or

**DNS Zones - Create DNS Zone**
Go to Service -> DNS Zones
Subscription: StackSimplify-Paid-Subscription (You need to have a paid subscription for this)
Resource Group: dns-zones
Name: rajdevopslearn.online
Resource Group Location: East US
Click on Review + Create


# Create Managed Identity

*Create MSI - Managed Service Identity for External DNS to access Azure DNS Zones*

Create Manged Service Identity (MSI)

Go to All Services -> Managed Identities -> Add
Resource Name: <ResourceName>
Subscription: Pay-as-you-go
Resource group: <ResourceGroup>
Location: <Location>
Click on Create

*Add Azure Role Assignment in MSI*

Opem MSI -> <ResourceName>
Click on Azure Role Assignments -> Add role assignment
Scope: Resource group
Subscription: Pay-as-you-go
Resource group: dns-rg
Role: Contributor
Make a note of Client Id and update in azure.json
Go to Overview -> Make a note of **Client ID"
Update in azure.json value for userAssignedIdentityID

*Associate MSI in AKS Cluster VMSS*

Go to All Services -> Virtual Machine Scale Sets (VMSS) -> Open aksdemo1 related VMSS (aks-agentpool-**********-vmss)

Go to Settings -> Identity -> User assigned -> Add -> <ResourceName>

# creating secret config from file

cd Ingress-ExternalDNS-AKS\External-DNS
kubectl create secret generic azure-config-file --from-file=azure.json

*To apply from the root manifest*

kubeclt apply -f -R ./


