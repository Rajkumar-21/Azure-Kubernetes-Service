# Ingress Basic implementation

* We are going to create a Static Public IP for Ingress in Azure AKS
* Associate that Public IP to Ingress Controller during installation.
* We are going to create a namespace ingress-basic for Ingress Controller where all ingress controller related things will be placed.
* In future, we install cert-manager for SSL certificates also in same namespace.
    Caution Note: This namespace is for Ingress controller stuff, ingress resource we can create in any other namespaces and not an issue. Only condition is create ingress resource and ingress pointed application in same namespace (Example: App1 and Ingress resource of App1 should be in same namespace)
* Create / Review Ingress Manifest
* Deploy a simple Nginx App1 with Ingress manifest and test it

# Create Static Public IP
$RESOURCE_GROUP=aks-rg
$LOCATION=westus
$AKS_ClUSTERNAME=myAKSCluster

az group create -n $RESOURCE_GROUP -l $LOCATION

az aks create -n $AKS_ClUSTERNAME -g $RESOURCE_GROUP -l $LOCATION

$NODE_ResourceGroup=az aks show --resource-group $RESOURCE_GROUP --name $AKS_ClUSTERNAME --query nodeResourceGroup -o tsv

$INGRESS_PUBLICIP=az network public-ip create --resource-group $NODE_ResourceGroup --name myAKSPublicIPForIngress --sku Standard --allocation-method static --query publicIp.ipAddress -o tsv

# Installing Ingrss Using Helm

kubectl create namespace ingress-ns

**Add the official stable repository**
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

**Customizing the Chart Before Installing**

helm show values ingress-nginx/ingress-nginx

**Use Helm to deploy an NGINX ingress controller**

helm install ingress-nginx ingress-nginx/ingress-nginx \
    --namespace ingress-ns \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."kubernetes\.io/os"=linux \
    --set controller.service.externalTrafficPolicy=Local \
    --set controller.service.loadBalancerIP=$INGRESS_PUBLICIP

**List Services with labels**

kubectl get service -l app.kubernetes.io/name=ingress-nginx --namespace ingress-ns

**List Pods**

kubectl get pods -n ingress-ns
kubectl get all -n ingress-ns 