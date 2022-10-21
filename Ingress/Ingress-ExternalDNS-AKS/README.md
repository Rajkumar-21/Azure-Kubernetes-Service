# Application gateway ingress controller add-on for an existing AKS cluster with an existing application gateway
$RESOURCE_GROUP="aks-rg"
$LOCATION="westus"
$AKS_ClUSTERNAME="myAKSCluster"

*Create a DNS Zone*

az network dns zone create -g $RESOURCE_GROUP -n rajdevopslearn.online


az group create -n $RESOURCE_GROUP -l $LOCATION

az aks create -n $AKS_ClUSTERNAME -g $RESOURCE_GROUP -l $LOCATION --network-plugin azure --enable-managed-identity

# Creating App Gateway and public IP

az network public-ip create -n myPublicIp -g $RESOURCE_GROUP --allocation-method Static --sku Standard

az network vnet create -n myVnet -g $RESOURCE_GROUP --address-prefix 10.0.0.0/16 --subnet-name mySubnetA --subnet-prefix 10.0.0.0/24

az network application-gateway create -n myApplicationGateway -l $LOCATION -g $RESOURCE_GROUP --sku Standard_v2 --public-ip-address myPublicIp --vnet-name myVnet --subnet mySubnetA --priority 100

$appgwId=az network application-gateway show -n myApplicationGateway -g $RESOURCE_GROUP -o tsv --query "id"

az aks enable-addons -n $AKS_ClUSTERNAME -g $RESOURCE_GROUP -a ingress-appgw --appgw-id $appgwId


**or**

# Create an ingress controller

```
# Create a namespace for ingress resources
kubectl create namespace ingress-basic
# Add the Helm repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
# Use Helm to deploy an NGINX ingress controller
helm install ingress-nginx ingress-nginx/ingress-nginx \
    --namespace ingress-basic \
    --set controller.replicaCount=2
```


# Peer the two virtual networks together

Since you deployed the AKS cluster in its own virtual network and the Application gateway in another virtual network, you'll need to peer the two virtual networks together in order for traffic to flow from the Application gateway to the pods in the cluster. Peering the two virtual networks requires running the Azure CLI command two separate times, to ensure that the connection is bi-directional. The first command will create a peering connection from the Application gateway virtual network to the AKS virtual network; the second command will create a peering connection in the other direction.

$nodeResourceGroup=az aks show -n $AKS_ClUSTERNAME -g $RESOURCE_GROUP -o tsv --query "nodeResourceGroup"

$aksVnetName=az network vnet list -g $nodeResourceGroup -o tsv --query "[0].name"

$aksVnetId=az network vnet show -n $aksVnetName -g $nodeResourceGroup -o tsv --query "id"

az network vnet peering create -n AppGWtoAKSVnetPeering -g $RESOURCE_GROUP --vnet-name myVnet --remote-vnet $aksVnetId --allow-vnet-access

$appGWVnetId=az network vnet show -n myVnet -g $RESOURCE_GROUP -o tsv --query "id"

az network vnet peering create -n AKStoAppGWVnetPeering -g $nodeResourceGroup --vnet-name $aksVnetName --remote-vnet $appGWVnetId --allow-vnet-access

or

# Create an ingress controller

```
# Create a namespace for ingress resources
kubectl create namespace ingress-basic
# Add the Helm repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
# Use Helm to deploy an NGINX ingress controller
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-basic --set controller.replicaCount=2
```

Setup Permissions

# Assign Variables
$tenantid=az account show --query tenantId -o tsv
$subscriptionid=az account show --query id -o tsv

$UserClientId=az aks show --name $AKS_ClUSTERNAME --resource-group $RESOURCE_GROUP --query identityProfile.kubeletidentity.clientId -o tsv

$DNSID=az network dns zone show --name rajdevopslearn.online --resource-group dnszone-rg --query id -o tsv

# Assign managed identity of cluster’s node pools DNS Zone Contributor rights on to Custom Domain DNS zone.
az role assignment create --assignee $UserClientId --role 'DNS Zone Contributor' --scope $DNSID

# Deploy ExternalDNS

```
# Add the Helm repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Use Helm to deploy an External DNS
helm install external-dns bitnami/external-dns --namespace ingress-basic --set provider=azure --set txtOwnerId=aksdemocluster --set policy=sync --set azure.resourceGroup=aksdemocluster-rg --set azure.tenantId=$tenantid --set azure.subscriptionId=$subscriptionid --set azure.useManagedIdentityExtension=true --set azure.userAssignedIdentityID=$UserClientId

```

# Installing cert-manager

# Label the ingress-basic namespace to disable resource validation
kubectl label namespace ingress-basic cert-manager.io/disable-validation=true

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart

```
helm install cert-manager jetstack/cert-manager --namespace ingress-basic --version v1.8.2 --set installCRDs=true
```
In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://cert-manager.io/docs/configuration/

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://cert-manager.io/docs/usage/ingress/
# Create a DNS Zone for custom Domain

**Using Azure CLI***
az network dns zone create -g dnszone-rg -n rajdevopslearn.online

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
Resource group: dnszone-rg
Role: Contributor
Make a note of Client Id and update in azure.json
Go to Overview -> Make a note of **Client ID"
Update in azure.json value for userAssignedIdentityID

*Associate MSI in AKS Cluster VMSS*

Go to All Services -> Virtual Machine Scale Sets (VMSS) -> Open aksdemo1 related VMSS (aks-agentpool-**********-vmss)

Go to Settings -> Identity -> User assigned -> Add -> <ResourceName>





# Verify Cert Manager pods
kubectl get pods --namespace ingress-basic

# Verify Cert Manager Services
kubectl get svc --namespace ingress-basic

# creating secret config from file

cd Ingress-ExternalDNS-AKS\External-DNS
kubectl create secret generic azure-config-file --from-file=azure.json

*To apply from the root manifest*

kubeclt apply -f -R ./


