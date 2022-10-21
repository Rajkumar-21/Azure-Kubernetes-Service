---
description: '- Rajkumar Ravi'
---

# AGIC in AKS

`az group create -n $RESOURCE_GROUP -l $LOCATION`

`az aks create -n $AKS_ClUSTERNAME -g $RESOURCE_GROUP -l $LOCATION --network-plugin azure --enable-managed-identity`

## Creating App Gateway and public IP

`az network public-ip create -n myPublicIp -g $RESOURCE_GROUP --allocation-method Static --sku Standard`

`az network vnet create -n myVnet -g $RESOURCE_GROUP --address-prefix 10.0.0.0/16 --subnet-name mySubnetA --subnet-prefix 10.0.0.0/24`

`az network application-gateway create -n myApplicationGateway -l $LOCATION -g $RESOURCE_GROUP --sku Standard_v2 --public-ip-address myPublicIp --vnet-name myVnet --subnet mySubnetA --priority 100`

`$appgwId=az network application-gateway show -n myApplicationGateway -g $RESOURCE_GROUP -o tsv --query "id"`

`az aks enable-addons -n $AKS_ClUSTERNAME -g $RESOURCE_GROUP -a ingress-appgw --appgw-id $appgwId`

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption><p>Variables</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>Creating RG and AKS Cluster</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption><p>Creating AG</p></figcaption></figure>

## Peer the two virtual networks together

Since you deployed the AKS cluster in its own virtual network and the Application gateway in another virtual network, you'll need to peer the two virtual networks together in order for traffic to flow from the Application gateway to the pods in the cluster. Peering the two virtual networks requires running the Azure CLI command two separate times, to ensure that the connection is bi-directional. The first command will create a peering connection from the Application gateway virtual network to the AKS virtual network; the second command will create a peering connection in the other direction.

`$nodeResourceGroup=az aks show -n $AKS_ClUSTERNAME -g $RESOURCE_GROUP -o tsv --query "nodeResourceGroup"`

`$aksVnetName=az network vnet list -g $nodeResourceGroup -o tsv --query "[0].name"`

`$aksVnetId=az network vnet show -n $aksVnetName -g $nodeResourceGroup -o tsv --query "id"`

`az network vnet peering create -n AppGWtoAKSVnetPeering -g $RESOURCE_GROUP --vnet-name myVnet --remote-vnet $aksVnetId --allow-vnet-access`

`$appGWVnetId=az network vnet show -n myVnet -g $RESOURCE_GROUP -o tsv --query "id"`

`az network vnet peering create -n AKStoAppGWVnetPeering -g $nodeResourceGroup --vnet-name $aksVnetName --remote-vnet $appGWVnetId --allow-vnet-access`

\-Installing cert-manager

## Label the ingress-basic namespace to disable resource validation

kubectl label namespace ingress-basic cert-manager.io/disable-validation=true

## Add the Jetstack Helm repository

helm repo add jetstack https://charts.jetstack.io

## Update your local Helm chart repository cache

helm repo update

## Install the cert-manager Helm chart

```
helm install cert-manager jetstack/cert-manager --namespace ingress-basic --version v1.8.2 --set installCRDs=true
```

In order to begin issuing certificates, you will need to set up a ClusterIssuer or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them can be found in our documentation:

https://cert-manager.io/docs/configuration/

For information on how to configure cert-manager to automatically provision Certificates for Ingress resources, take a look at the `ingress-shim` documentation:

https://cert-manager.io/docs/usage/ingress/

## Create a DNS Zone for custom Domain

**Using Azure CLI**\* az network dns zone create -g dnszone-rg -n rajdevopslearn.online

or

**DNS Zones - Create DNS Zone** Go to Service -> DNS Zones Subscription: StackSimplify-Paid-Subscription (You need to have a paid subscription for this) Resource Group: dns-zones Name: rajdevopslearn.online Resource Group Location: East US Click on Review + Create

## Create Managed Identity

_Create MSI - Managed Service Identity for External DNS to access Azure DNS Zones_

Create Manged Service Identity (MSI)

Go to All Services -> Managed Identities -> Add Resource Name: Subscription: Pay-as-you-go Resource group: Location: Click on Create

_Add Azure Role Assignment in MSI_

Opem MSI -> Click on Azure Role Assignments -> Add role assignment Scope: Resource group Subscription: Pay-as-you-go Resource group: dnszone-rg Role: Contributor Make a note of Client Id and update in azure.json Go to Overview -> Make a note of \*\*Client ID" Update in azure.json value for userAssignedIdentityID

_Associate MSI in AKS Cluster VMSS_

Go to All Services -> Virtual Machine Scale Sets (VMSS) -> Open aksdemo1 related VMSS (aks-agentpool-\*\*\*\*\*\*\*\*\*\*-vmss)

Go to Settings -> Identity -> User assigned -> Add ->

## creating secret config from file

`cd Ingress-ExternalDNS-AKS\External-DNS`

`kubectl create secret generic azure-config-file --from-file=azure.json`

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption><p>Creating Secret</p></figcaption></figure>

{% code title="azure.json" overflow="wrap" %}
```json
{
  "tenantId": "<tenantId>",
  "subscriptionId": "id",
  "resourceGroup": "dns-rg", 
  "useManagedIdentityExtension": true,
  "userAssignedIdentityID": "<clientId>"
}
```
{% endcode %}

{% code title="ExternalDNSfile.yml" overflow="wrap" %}
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: rajdevopslearn-sc
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: rajdevopslearn-clusterRole
rules:
- apiGroups: [""]
  resources: ["services","endpoints","pods", "nodes"]
  verbs: ["get","watch","list"]
- apiGroups: ["extensions","networking.k8s.io"]
  resources: ["ingresses"] 
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: rajdevopslearn-clusterRoleBinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: rajdevopslearn-clusterRole
subjects:
- kind: ServiceAccount
  name: rajdevopslearn-sc
  namespace: default
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rajdevopslearn
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: rajdevopslearn
  template:
    metadata:
      labels:
        app: rajdevopslearn
    spec:
      serviceAccountName: rajdevopslearn-sc
      containers:
      - name: rajdevopslearn
        image: k8s.gcr.io/external-dns/external-dns:v0.11.0
        args:
        - --source=service
        - --source=ingress
        #- --domain-filter=example.com # (optional) limit to only example.com domains; change to match the zone created above.
        - --provider=azure
        #- --azure-resource-group=externaldns # (optional) use the DNS zones from the specific resource group
        volumeMounts:
        - name: azure-config-file
          mountPath: /etc/kubernetes
          readOnly: true
      volumes:
      - name: azure-config-file
        secret:
          secretName: azure-config-file
```
{% endcode %}

{% code title="IngressService.yml" overflow="wrap" %}
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-rajdevopslearn.online
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - app1.rajdevopslearn.online          
    - app2.rajdevopslearn.online
    secretName: tls-secret
  rules:
    - host: app1.rajdevopslearn.online
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app1-nginx-clusterip-service
                port: 
                  number: 80
    - host: app2.rajdevopslearn.online
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app2-nginx-clusterip-service
                port: 
                  number: 80                                                           
                     
```
{% endcode %}

{% code title="app1-deployment.yml" overflow="wrap" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1-nginx-deployment
  labels:
    app: app1-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1-nginx
  template:
    metadata:
      labels:
        app: app1-nginx
    spec:
      containers:
        - name: app1-nginx
          image: stacksimplify/kube-nginxapp1:1.0.0
          ports:
            - containerPort: 80
```
{% endcode %}

{% code title="app1-service.yml" overflow="wrap" %}
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app1-nginx-clusterip-service
  labels:
    app: app1-nginx
spec:
  type: ClusterIP
  selector:
    app: app1-nginx
  ports:
    - port: 80
      targetPort: 80
```
{% endcode %}

{% code title="app2-deployment.yml" overflow="wrap" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2-nginx-deployment
  labels:
    app: app2-nginx 
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app2-nginx
  template:
    metadata:
      labels:
        app: app2-nginx
    spec:
      containers:
        - name: app2-nginx
          image: stacksimplify/kube-nginxapp2:1.0.0
          ports:
            - containerPort: 80
```
{% endcode %}

{% code title="app2-service.yml" overflow="wrap" %}
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app2-nginx-clusterip-service
  labels:
    app: app2-nginx
  annotations:
spec:
  type: ClusterIP
  selector:
    app: app2-nginx
  ports:
    - port: 80
      targetPort: 80
```
{% endcode %}

{% code title="certIssuer.yml" overflow="wrap" %}
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: rajkumarravi21897@gmail.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt
    solvers:
      - http01:
          ingress:
            class: nginx
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption><p>Applying Manifest files</p></figcaption></figure>
