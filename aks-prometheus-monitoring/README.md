
# Monitor Azure Kubernetes Service(AKS) with Prometheus and Grafana

***Reference-Gitbook***

[Monitor AKS with Prometheus and Grafana ](https://azuredevops.gitbook.io/kubernetes-in-microsoft-azure/) 

---
Add-ons-extensions-and-other-integrations-with-aks/monitor-aks-with-prometheus-and-grafana
Prometheus is an open-source systems monitoring and alerting toolkit. Prometheus collects and stores its metrics as time series data, i.e. metrics information is stored with the timestamp at which it was recorded, alongside optional key-value pairs called labels. Grafana is an open-source platform for monitoring and observability. It allows you to query, visualize, alert on and understand your metrics no matter where they are stored.

We will use Helm to install and manage Prometheus and Grafana on AKS cluster.

__Creating AKS Cluster__

```
$RESOURCE_GROUP="myaks-group"

$AKS_CLUSTER_NAME="myAKSCluster"

$LOCATION="eastus"

az group create -n $RESOURCE_GROUP -l $LOCATION

az aks create -n $AKS_CLUSTER_NAME -g $RESOURCE_GROUP -l $LOCATION

# To save the credentials to connect k8s

az aks get-credentials -n $AKS_CLUSTER_NAME -g $RESOURCE_GROUP
```

___Install Prometheus and Grafana___

```
# Define public Kubernetes chart repository in the Helm configuration

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Update local repositories
helm repo update

# Search for newly installed repositories
helm repo list

# Create a namespace for Prometheus and Grafana resources
kubectl create ns prometheus

# Install Prometheus using HELM
helm install prometheus prometheus-community/kube-prometheus-stack -n prometheus

# Check all resources in Prometheus Namespace
kubectl get all -n prometheus

```
To map the port of the pod to check the Prometheus dashboard otherwise you can create a service to expose the pod
```
kubectl port-forward -n prometheus prometheus-prometheus-monitor-kube-pr-prometheus-0 9090
```

In order to login to the Grafana dashboard, username and password are required which can be retrieved by using the below command:
# Get the Username
```
kubectl get secret -n prometheus <prometheus-grafana-podname> -o=jsonpath='{.data.admin-user}' |base64 -d
```

# Get the Password
```
kubectl get secret -n prometheus <prometheus-grafana-podname> -o=jsonpath='{.data.admin-password}' |base64 -d
```
Creating service to expose the pod to access it
Here im using load balancer to check and access the Grafana
```
kubectl expose pod prometheus-monitor-grafana-7d59fd4d54-ffjf8 --port=3000 --type=LoadBalancer -n prometheus
```

# Create Service principle to add the Data Source in Grafana

```
az ad sp create-for-rbac --name prom-spn--role="Monitoring Reader" --scopes="/subscriptions/my-subscription-id/resourceGroups/myaks-group"
```
Save the password and appId and tenantId details to use in Data Source - Grafana
Login Grafana dashboard from the credentials got from the previous task in decoding the grafana secrets

Now click on Add your first data source
Then select Azure Monitor and pass the SPN details to configure it
Now we are connected our Azure Monitor to the Grafana
Also, we can use inbuilt dashboards to view our details

