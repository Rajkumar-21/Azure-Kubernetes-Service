---
description: 'Source: Microsoft'
---

# Monitor AKS with Prometheus and Grafana

[Prometheus](https://prometheus.io/) is a popular open-source metric monitoring solution and is a part of the [Cloud Native Compute Foundation](https://www.cncf.io/). Container insights provides a seamless onboarding experience to collect Prometheus metrics.

Typically, to use Prometheus, you need to set up and manage a Prometheus server with a store. If you integrate with Azure Monitor, a Prometheus server isn't required. You only need to expose the Prometheus metrics endpoint through your exporters or pods (application). Then the containerized agent for Container insights can scrape the metrics for you.

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

**Creating AKS cluster if there is no existing cluster**

Prometheus is an open-source systems monitoring and alerting toolkit. Prometheus collects and stores its metrics as time series data, i.e. metrics information is stored with the timestamp at which it was recorded, alongside optional key-value pairs called labels. Grafana is an open-source platform for monitoring and observability. It allows you to query, visualize, alert on and understand your metrics no matter where they are stored.

```powershell
$RESOURCE_GROUP="myaks-group"

$AKS_CLUSTER_NAME="myAKSCluster"

$LOCATION="eastus"

az group create -n $RESOURCE_GROUP -l $LOCATION

az aks create -n myAKSCluster -g $RESOURCE_GROUP -l $LOCATION
```

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>Creating AKS cluster</p></figcaption></figure>

`az aks get-credentials -n $AKS_CLUSTER_NAME -g $RESOURCE_GROUP`

Configure the credentials to access the k8s

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

**Install Prometheus using helm chart**

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

<figure><img src="../.gitbook/assets/image (20).png" alt="Install Prometheus using helm chart"><figcaption><p><strong>Installing Prometheus using helm chart</strong></p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

To map the port of the pod to check the Prometheus dashboard otherwise you can create a service to expose the pod

`kubectl port-forward -n prometheus prometheus-prometheus-monitor-kube-pr-prometheus-0 9090`

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption><p>kubectl get secret -n prometheus</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption><p>kubectl describe secret prometheus-monitor-grafana -n prometheus</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>kubectl get secret prometheus-monitor-grafana -n prometheus -o yaml</p></figcaption></figure>

In order to login to the Grafana dashboard, username and password are required which can be retrieved by using the below command:

{% code overflow="wrap" %}
```
# Get the Username
kubectl get secret -n prometheus <prometheus-grafana-podname> -o=jsonpath='{.data.admin-user}' |base64 -d

# Get the Password
kubectl get secret -n prometheus <prometheus-grafana-podname> -o=jsonpath='{.data.admin-password}' |base64 -d
```
{% endcode %}

**Creating service to expose the pod to access it**

Here im using load balancer to check and access the Grafana

`kubectl expose pod prometheus-monitor-grafana-7d59fd4d54-ffjf8 --port=3000 --type=`LoadBalancer `-n prometheus`

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

**Create Service principle to add the Data Source in Grafana**

`az ad sp create-for-rbac --name prom-spn--role="Monitoring Reader" --scopes="/subscriptions/my-subscription-id/resourceGroups/myaks-group"`

_Save the password and appId and tenantId details to use in Data Source - Grafana_

Login Grafana dashboard from the credentials got from the previous task in decoding the grafana secrets

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Now click on _Add your first data source_

Then select Azure Monitor and pass the SPN details to configure it

Now we are connected our Azure Monitor to the Grafana

Also, we can use inbuilt dashboards to view our details

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

