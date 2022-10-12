# Steps to implement ***kv2k8s***

```$KEY_VAULT_NAME="mykeyvault210"```

```$RESOURCE_GROUP="myaks-group"```

```$AKS_CLUSTER_NAME="myAKSCluster"```

```$LOCATION="eastus"```

```$KEY_VAULT_SECRET_NAME="mkeyvaulty"```

```$SUBSCRIPTION_ID=az account show --query id```

```$TENANT_ID=az account show --query tenantId```

```$IDENTITY_NAME=myaks-id```

---
# AKS Creation using CLI

```az group create -n $RESOURCE_GROUP -l $LOCATION```

```az aks create -n myAKSCluster -g $RESOURCE_GROUP -l $LOCATION --enable-addons azure-keyvault-secrets-provider --enable-managed-identity --enable-pod-identity --enable-pod-identity-with-kubenet```

---

# Keyvault Creation

```az keyvault create -n $KEY_VAULT_NAME -g $RESOURCE_GROUP -l $LOCATION```

```az keyvault secret set --vault-name $KEY_VAULT_NAME -n aksSecret --value MyAKSSecrets```
```az keyvault secret set --vault-name $KEY_VAULT_NAME -n username --value admin_user```
```az keyvault secret set --vault-name $KEY_VAULT_NAME -n password --value admin_password```

# Identity Creation
---

```az group create --name myIdentityResourceGroup --location $LOCATION
$IDENTITY_RESOURCE_GROUP="myIdentityResourceGroup"```

```az identity create --name $IDENTITY_NAME --resource-group $IDENTITY_RESOURCE_GROUP --location $LOCATION --subscription $SUBSCRIPTION_ID```

```$IDENTITY_CLIENT_ID=az identity show -g $IDENTITY_RESOURCE_GROUP -n $IDENTITY_NAME --query clientId -otsv```
```$IDENTITY_RESOURCE_ID=az identity show -g $IDENTITY_RESOURCE_GROUP -n $IDENTITY_NAME --query id -otsv```

---
# Node Group
```$NODE_GROUP=az aks show -g $RESOURCE_GROUP -n $AKS_CLUSTER_NAME --query nodeResourceGroup -o tsv```
```$NODE_RESOURCE_ID=az group show -n $NODE_GROUP -o tsv --query "id"```

# Assign permissions for the managed identity
***
The managed identity that will be assigned to the pod needs to be granted permissions that align with the actions it will be taking.

To run the demo, the IDENTITY_CLIENT_ID managed identity must have Virtual Machine Contributor permissions in the resource group that contains the virtual machine scale set of your AKS cluster.
***

```az role assignment create --role "Virtual Machine Contributor" --assignee "$IDENTITY_CLIENT_ID" --scope $NODES_RESOURCE_ID```
---

#Create a pod identity
---

```$POD_IDENTITY_NAME="myaks-pod-id"```
```$POD_IDENTITY_NAMESPACE="csi"```
```az aks pod-identity add --resource-group $RESOURCE_GROUP --cluster-name $AKS_CLUSTER_NAME --namespace ${POD_IDENTITY_NAMESPACE}  --name ${POD_IDENTITY_NAME} --identity-resource-id ${IDENTITY_RESOURCE_ID}```
***If pod identity not defined and this will use for existing cluster***
# Troubleshoots
```az aks update -g $RESOURCE_GROUP -n $AKS_CLUSTER_NAME --enable-managed-identity```
```az aks update -g $RESOURCE_GROUP -n $AKS_CLUSTER_NAME --enable-pod-identity --enable-pod-identity-with-kubenet```
---
# List Identities

```kubectl get azureidentity -n $POD_IDENTITY_NAMESPACE```
```kubectl get azureidentitybinding -n $POD_IDENTITY_NAMESPACE```

```az keyvault set-policy --name mykeyvault21970 --object-id "56a62cf4-a155-4958-ab47-9fdc55059c93" --secret-permissions get list```

az aks show --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER_NAME --query "oidcIssuerProfile.issuerUrl" -otsv


