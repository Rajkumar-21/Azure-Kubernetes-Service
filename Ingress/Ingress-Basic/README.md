# Ingress Basic implementation

* We are going to create a Static Public IP for Ingress in Azure AKS
* Associate that Public IP to Ingress Controller during installation.
* We are going to create a namespace ingress-basic for Ingress Controller where all ingress controller related things will be placed.
* In future, we install cert-manager for SSL certificates also in same namespace.
    Caution Note: This namespace is for Ingress controller stuff, ingress resource we can create in any other namespaces and not an issue. Only condition is create ingress resource and ingress pointed application in same namespace (Example: App1 and Ingress resource of App1 should be in same namespace)
* Create / Review Ingress Manifest
* Deploy a simple Nginx App1 with Ingress manifest and test it

