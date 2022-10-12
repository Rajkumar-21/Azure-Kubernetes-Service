You can use ReadinessProbe and LivenessProbe together in the same Pod. There is nothing wrong with that since ReadinessProbe is here for you to prevent a Pod from joining a service before being ready to serve traffic, and LivenessProbe is here to kill a Pod that has become unhealthy.

So, they are configured almost the same way—they don't have the exact same purpose, and it is fine to use them together. Please note that both probes share these parameters:

initialDelaySeconds: The number of seconds to wait before the first probe execution
periodSeocnds: The number of seconds between two probes
timeoutSeconds: The number of seconds to wait before timeout
successThreshold: The number of successful attempts to consider a Pod is ready (for ReadinessProbe) or healthy (for LivenessProbe)
failureThreshold : The number of failed attempts to consider a Pod is not ready (for ReadinessProbe) or ready to be killed (for LivenessProbe)

# Securing your Pods using the NetworkPolicy object
The NetworkPolicy object is the last resource kind we need to discover as part of this chapter to have an overview of services in this chapter. NetworkPolicy will allow you to define network firewalls directly implemented in your cluster.

# Why do you need NetworkPolicy?
When you have to manage a real Kubernetes workload in production, you'll have to deploy more and more applications onto it, and it is possible that these applications will have to communicate with each other.

Achieving communication between applications is really one of the fundamental objectives of a microservice architecture. Most of this communication will be done through the network, and the network is forcibly something that you want to secure by using firewalls.

Kubernetes has its own implementation of network firewalls called NetworkPolicy. This is a new resource kind we are going to discover. Say that you want one nginx resource to be accessible on port 80 from a particular IP address and to block any other traffic that doesn't match these requirements. To do that, you'll need to use NetworkPolicy and attach it to that Pod.

NetworkPolicy brings three benefits, as follows:

You can build egress/ingress rules based on Classless Inter-Domain Routing (CIDR) blocks.
You can build egress/ingress rules based on Pods labels and selectors (just as we've seen before with services' and Pods' association).
You can build egress/ingress rules based on namespaces