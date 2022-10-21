# Integrating Azure Key Vault in AKS

## Identity Access Modes

The Azure Key Vault Provider offers four modes for accessing a Key Vault instance

### Best Practices <a href="#best-practices" id="best-practices"></a>

Following order of access modes is recommended for Secret Store CSI driver AKV provider:

| Access Option                                          | Comment                                                                                                                                                                                                        |
| ------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Workload Identity (Preview)                            | This is currently in preview. It’s a secure way to access Key Vault based on the [Workload Identity Federation](https://docs.microsoft.com/en-us/azure/active-directory/develop/workload-identity-federation). |
| Pod Identity                                           | This is the most secure way to get access to Azure resources (AKV in this case) as it uses the managed identity bound to the Pod.                                                                              |
| Managed Identities (System-assigned and User-assigned) | Managed identities eliminate the need for developers to manage credentials. Managed identities provide an identity for applications to use when connecting to Azure Keyvault.                                  |
| Service Principal                                      | This is the last option to consider while connecting to AKV as access credentials need to be created as Kubernetes Secret and stored in plain text in etcd.                                                    |

***

[**Service Principal**](https://azure.github.io/secrets-store-csi-driver-provider-azure/docs/configurations/identity-access-modes/service-principal-mode/)

Use a Service Principal to access Keyvault.

[**Workload Identity (Preview)**](https://azure.github.io/secrets-store-csi-driver-provider-azure/docs/configurations/identity-access-modes/workload-identity-mode/)

Use Workload Identity to access Keyvault.

[**Pod Identity**](https://azure.github.io/secrets-store-csi-driver-provider-azure/docs/configurations/identity-access-modes/pod-identity-mode/)

Use Pod Identity to access Keyvault.

[**User-assigned Managed Identity**](https://azure.github.io/secrets-store-csi-driver-provider-azure/docs/configurations/identity-access-modes/user-assigned-msi-mode/)

Use a User-assigned Managed Identity to access Keyvault.

[**System-assigned Managed Identity**](https://azure.github.io/secrets-store-csi-driver-provider-azure/docs/configurations/identity-access-modes/system-assigned-msi-mode/)

Use a System-assigned Managed Identity to access Keyvault.

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Be sure that a Secrets Store CSI Driver pod and a Secrets Store Provider Azure pod are running on each node in your cluster's node pools.

### Upgrade an existing AKS cluster with Azure Key Vault Provider for Secrets Store CSI Driver support <a href="#upgrade-an-existing-aks-cluster-with-azure-key-vault-provider-for-secrets-store-csi-driver-support" id="upgrade-an-existing-aks-cluster-with-azure-key-vault-provider-for-secrets-store-csi-driver-support"></a>

To upgrade an existing AKS cluster with Azure Key Vault Provider for Secrets Store CSI Driver capability, use the [az aks enable-addons](https://learn.microsoft.com/en-us/cli/azure/aks#az-aks-enable-addons) command with the `azure-keyvault-secrets-provider` add-on:

{% code overflow="wrap" %}
```powershell
az aks enable-addons --addons azure-keyvault-secrets-provider --name myAKSCluster --resource-group myResourceGroup
```
{% endcode %}

### Create or use an existing Azure key vault <a href="#create-or-use-an-existing-azure-key-vault" id="create-or-use-an-existing-azure-key-vault"></a>

In addition to an AKS cluster, you'll need an Azure key vault resource that stores the secret content. Keep in mind that the key vault's name must be globally unique.

Your Azure key vault can store keys, secrets, and certificates. In this example, you'll set a plain-text secret called `ExampleSecret`:

Creating Secret in the existing keyvault and saving in the keyvault.

`az keyvault secret set --vault-name mykeyvault1230 -n mkeyvaulty --value MyAKSExampleSecret`

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Creating Pod-Identity based _**AK2K8S**_

{% code title="Instructions" overflow="wrap" lineNumbers="true" %}
```markdown
Steps to implement ***kv2k8s***

$KEY_VAULT_NAME="mykeyvault210"
$RESOURCE_GROUP="myaks-group"
$AKS_CLUSTER_NAME="myAKSCluster"
$LOCATION="eastus"
$KEY_VAULT_SECRET_NAME="mkeyvaulty"
$SUBSCRIPTION_ID=az account show --query id
$TENANT_ID=az account show --query tenantId
$IDENTITY_NAME=myaks-id

---

az group create -n $RESOURCE_GROUP -l $LOCATION
az aks create -n myAKSCluster -g $RESOURCE_GROUP -l $LOCATION --enable-addons azure-keyvault-secrets-provider --enable-managed-identity --enable-pod-identity --enable-pod-identity-with-kubenet

# Keyvault Creation

az keyvault create -n $KEY_VAULT_NAME -g $RESOURCE_GROUP -l $LOCATION

az keyvault secret set --vault-name $KEY_VAULT_NAME -n aksSecret --value MyAKSSecrets
az keyvault secret set --vault-name $KEY_VAULT_NAME -n username --value admin_user
az keyvault secret set --vault-name $KEY_VAULT_NAME -n password --value admin_password

---
# Identity Creation
az group create --name myIdentityResourceGroup --location $LOCATION
$IDENTITY_RESOURCE_GROUP="myIdentityResourceGroup"
az identity create --name $IDENTITY_NAME --resource-group $IDENTITY_RESOURCE_GROUP --location $LOCATION --subscription $SUBSCRIPTION_ID

$IDENTITY_CLIENT_ID=az identity show -g $IDENTITY_RESOURCE_GROUP -n $IDENTITY_NAME --query clientId -otsv
$IDENTITY_RESOURCE_ID=az identity show -g $IDENTITY_RESOURCE_GROUP -n $IDENTITY_NAME --query id -otsv

---
# Node Group
$NODE_GROUP=az aks show -g $RESOURCE_GROUP -n $AKS_CLUSTER_NAME --query nodeResourceGroup -o tsv
$NODE_RESOURCE_ID=az group show -n $NODE_GROUP -o tsv --query "id"

# Assign permissions for the managed identity
***
The managed identity that will be assigned to the pod needs to be granted permissions that align with the actions it will be taking.

To run the demo, the IDENTITY_CLIENT_ID managed identity must have Virtual Machine Contributor permissions in the resource group that contains the virtual machine scale set of your AKS cluster.
***

az role assignment create --role "Virtual Machine Contributor" --assignee "$IDENTITY_CLIENT_ID" --scope $NODES_RESOURCE_ID

# Create a pod identity

$POD_IDENTITY_NAME="myaks-pod-id"
$POD_IDENTITY_NAMESPACE="csi"
az aks pod-identity add --resource-group $RESOURCE_GROUP --cluster-name $AKS_CLUSTER_NAME --namespace ${POD_IDENTITY_NAMESPACE}  --name ${POD_IDENTITY_NAME} --identity-resource-id ${IDENTITY_RESOURCE_ID}
***If pod identity not defined and this will use for existing cluster***
# Troubleshoots
az aks update -g $RESOURCE_GROUP -n $AKS_CLUSTER_NAME --enable-managed-identity
az aks update -g $RESOURCE_GROUP -n $AKS_CLUSTER_NAME --enable-pod-identity --enable-pod-identity-with-kubenet

---
# List Identities

kubectl get azureidentity -n $POD_IDENTITY_NAMESPACE
kubectl get azureidentitybinding -n $POD_IDENTITY_NAMESPACE

az keyvault set-policy --name mykeyvault21970 --object-id "<object ID>" --secret-permissions get list



```
{% endcode %}

Now create k8s manifest to integrate key vault provider to pod

{% code title="namespace.yml" %}
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name:  csi
```
{% endcode %}

{% code title="pod.yml" overflow="wrap" lineNumbers="true" %}
```yaml
# This is a sample pod definition for using SecretProviderClass and aad-pod-identity to access the key vault
kind: Pod
apiVersion: v1
metadata:
  name: busybox-secrets-store-inline-podid
  namespace: csi
  labels:
    aadpodidbinding: aadpodid                   # Set the label value to the name of your pod identity
spec:
  containers:
    - name: busybox
      image: k8s.gcr.io/e2e-test-images/busybox:1.29-1
      resources:
      limits:
        memory: 512Mi
        cpu: "1"
      requests:
        memory: 256Mi
        cpu: "0.2"
      command:
        - "/bin/sleep"
        - "10000"
      volumeMounts:
      - name: secrets-store01-inline
        mountPath: "/mnt/secrets-store"
        readOnly: true
  volumes:
    - name:  secrets-store01-inline
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: "aks_kv_spc"
```
{% endcode %}

{% code title="secretproviderclass.yml" overflow="wrap" %}
```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: aks_kv_spc
  namespace: csi
spec:
  provider: azure
  parameters:
    usePodIdentity: "true"               # Set to true for using aad-pod-identity to access your key vault
    keyvaultName: mykeyvault21897       # Set to the name of your key vault
    cloudName: ""                        # [OPTIONAL for Azure] if not provided, the Azure environment defaults to AzurePublicCloud
    objects:  |
      array:
        - |
          objectName: aksSecret
          objectType: secret             # object types: secret, key, or cert
          objectVersion: ""              # [OPTIONAL] object versions, default to latest if empty
    tenantId: "<my-tenant>"                # The tenant ID of the key vault
```
{% endcode %}

Now cross check the containers with the mounted volumes to check the secrets are mapped and mounted correctly.

{% embed url="https://github.com/Rajkumar-21/Azure-Kubernetes-Service/tree/main/aks-csi-keyvault" %}
GitHub for this project
{% endembed %}
