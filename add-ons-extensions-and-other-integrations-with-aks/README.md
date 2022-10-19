---
description: 'Source: https://learn.microsoft.com/en-us/azure/aks/integrations'
---

# Add-ons, extensions and other integrations with AKS

### Add-ons <a href="#add-ons" id="add-ons"></a>

Add-ons are a fully supported way to provide extra capabilities for your AKS cluster. Add-ons' installation, configuration, and lifecycle is managed by AKS. Use `az aks addon` to install an add-on or manage the add-ons for your cluster.

The following rules are used by AKS for applying updates to installed add-ons:

* Only an add-on's patch version can be upgraded within a Kubernetes minor version. The add-on's major/minor version will not be upgraded within the same Kubernetes minor version.
* The major/minor version of the add-on will only be upgraded when moving to a later Kubernetes minor version.
* Any breaking or behavior changes to the add-on will be announced well before, usually 60 days, for a GA minor version of Kubernetes on AKS.
* Add-ons can be patched weekly with every new release of AKS which will be announced in the release notes. AKS releases can be controlled using [maintenance windows](https://learn.microsoft.com/en-us/azure/aks/planned-maintenance) and followed using [release tracker](https://learn.microsoft.com/en-us/azure/aks/release-tracker).

#### Exceptions

* Add-ons will be upgraded to a new major/minor version (or breaking change) within a Kubernetes minor version if either the cluster's Kubernetes version or the add-on version are in preview.
* It is also possible, in unavoidable circumstances such as CVE security patches or critical bug fixes, that there may be times when an add-on needs to be updated within a GA minor version.

#### Available add-ons

The below table shows the available add-ons.

| Name                            | Description                                                                                                                        | More details                                                                                                                                                                    |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| http\_application\_routing      | Configure ingress with automatic public DNS name creation for your AKS cluster.                                                    | [HTTP application routing add-on on Azure Kubernetes Service (AKS)](https://learn.microsoft.com/en-us/azure/aks/http-application-routing)                                       |
| monitoring                      | Use Container Insights monitoring with your AKS cluster.                                                                           | [Container insights overview](https://learn.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-overview)                                                     |
| virtual-node                    | Use virtual nodes with your AKS cluster.                                                                                           | [Use virtual nodes](https://learn.microsoft.com/en-us/azure/aks/virtual-nodes)                                                                                                  |
| azure-policy                    | Use Azure Policy for AKS, which enables at-scale enforcements and safeguards on your clusters in a centralized, consistent manner. | [Understand Azure Policy for Kubernetes clusters](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/policy-for-kubernetes#install-azure-policy-add-on-for-aks) |
| ingress-appgw                   | Use Application Gateway Ingress Controller with your AKS cluster.                                                                  | [What is Application Gateway Ingress Controller?](https://learn.microsoft.com/en-us/azure/application-gateway/ingress-controller-overview)                                      |
| open-service-mesh               | Use Open Service Mesh with your AKS cluster.                                                                                       | [Open Service Mesh AKS add-on](https://learn.microsoft.com/en-us/azure/aks/open-service-mesh-about)                                                                             |
| azure-keyvault-secrets-provider | Use Azure Keyvault Secrets Provider addon.                                                                                         | [Use the Azure Key Vault Provider for Secrets Store CSI Driver in an AKS cluster](https://learn.microsoft.com/en-us/azure/aks/csi-secrets-store-driver)                         |
| web\_application\_routing       | Use a managed NGINX ingress Controller with your AKS cluster.                                                                      | [Web Application Routing Overview](https://learn.microsoft.com/en-us/azure/aks/web-app-routing)                                                                                 |
| keda                            | Event-driven autoscaling for the applications on your AKS cluster.                                                                 | [Simplified application autoscaling with Kubernetes Event-driven Autoscaling (KEDA) add-on](https://learn.microsoft.com/en-us/azure/aks/keda-about)                             |

### Extensions

Cluster extensions build on top of certain Helm charts and provide an Azure Resource Manager-driven experience for installation and lifecycle management of different Azure capabilities on top of your Kubernetes cluster. For more details on the specific cluster extensions for AKS, see [Deploy and manage cluster extensions for Azure Kubernetes Service (AKS)](https://learn.microsoft.com/en-us/azure/aks/cluster-extensions?tabs=azure-cli). For more details on the currently available cluster extensions, see [Currently available extensions](https://learn.microsoft.com/en-us/azure/aks/cluster-extensions?tabs=azure-cli#currently-available-extensions).

### Difference between extensions and add-ons

Both extensions and add-ons are supported ways to add functionality to your AKS cluster. When you install an add-on, the functionality is added as part of the AKS resource provider in the Azure API. When you install an extension, the functionality is added as part of a separate resource provider in the Azure API.



### Open source and third-party integrations <a href="#open-source-and-third-party-integrations" id="open-source-and-third-party-integrations"></a>

You can install many open source and third-party integrations on your AKS cluster, but these open-source and third-party integrations are not covered by the [AKS support policy](https://learn.microsoft.com/en-us/azure/aks/support-policies).

The below table shows a few examples of open-source and third-party integrations.

| Name                                      | Description                                                                                               | More details                                                                                                                                                                                                                           |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Helm](https://helm.sh/)                  | An open-source packaging tool that helps you install and manage the lifecycle of Kubernetes applications. | [Quickstart: Develop on Azure Kubernetes Service (AKS) with Helm](https://learn.microsoft.com/en-us/azure/aks/quickstart-helm)                                                                                                         |
| [Prometheus](https://prometheus.io/)      | An open source monitoring and alerting toolkit.                                                           | [Container insights with metrics in Prometheus format](https://learn.microsoft.com/en-us/azure/aks/monitor-aks#container-insights), [Prometheus Helm chart](https://github.com/prometheus-community/helm-charts#usage)                 |
| [Grafana](https://grafana.com/)           | An open-source dashboard for observability.                                                               | [Deploy Grafana on Kubernetes](https://grafana.com/docs/grafana/latest/installation/kubernetes/) or use [Managed Grafana](https://learn.microsoft.com/en-us/azure/managed-grafana/overview)                                            |
| [Couchbase](https://www.couchbase.com/)   | A distributed NoSQL cloud database.                                                                       | [Install Couchbase and the Operator on AKS](https://docs.couchbase.com/operator/current/tutorial-aks.html)                                                                                                                             |
| [OpenFaaS](https://www.openfaas.com/)     | An open-source framework for building serverless functions by using containers.                           | [Use OpenFaaS with AKS](https://learn.microsoft.com/en-us/azure/aks/openfaas)                                                                                                                                                          |
| [Apache Spark](https://spark.apache.org/) | An open source, fast engine for large-scale data processing.                                              | Running Apache Spark jobs requires a minimum node size of _Standard\_D3\_v2_. See [running Spark on Kubernetes](https://spark.apache.org/docs/latest/running-on-kubernetes.html) for more details on running Spark jobs on Kubernetes. |
| [Istio](https://istio.io/)                | An open-source service mesh.                                                                              | [Istio Installation Guides](https://istio.io/latest/docs/setup/install/)                                                                                                                                                               |
| [Linkerd](https://linkerd.io/)            | An open-source service mesh.                                                                              | [Linkerd Getting Started](https://linkerd.io/getting-started/)                                                                                                                                                                         |
| [Consul](https://www.consul.io/)          | An open source, identity-based networking solution.                                                       | [Getting Started with Consul Service Mesh for Kubernetes](https://learn.hashicorp.com/tutorials/consul/service-mesh-deploy)                                                                                                            |
