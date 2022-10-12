# Introduction to Kubernetes namespaces
The more applications you deploy on your Kubernetes clusters, the more you will feel the need to keep your cluster organized. You can already use labels and annotations to tidy up the objects inside your cluster, but you can go even further by using namespaces, which are presented by Kubernetes as virtual clusters within your cluster.

Using namespaces in Kubernetes allows you to split your cluster into smaller parts that will be logically isolated. Once namespaces have been created, you can launch Kubernetes objects such as Pods, which will only exist in that namespace. So, all the operations that are run against the cluster with kubectl will be scoped to that individual namespace: you can perform as many operations as possible while eliminating the risk of impacting resources that are in another namespace.

*Lastly, namespaces allow you to define resources limits and quotas for the Pods running in them.*

**Why do you need namespaces?**

As we mentioned previously, namespaces in Kubernetes are a way to help the cluster administrator keep everything clean and organized, while providing resource isolation when running kubectl commands.

The biggest Kubernetes clusters can run hundreds or even thousands of applications in Pods. Very often, those Pods come with services, ConfigMaps, volumes, and more. When everything is deployed in the same namespace, it can become very complex to know which particular resource belongs to which application.

If, by misfortune, you update the wrong resource using kubectl, you might end up breaking an app running in your cluster. To resolve that, you can use labels and selectors but even then, as the number of resources grows, managing the cluster will quickly become chaotic if you don't start using namespaces.

**How namespaces are used to split resources into chunks**


Right after you've installed Kubernetes, when your cluster is brand new, it is created with namespaces. So, even if you didn't notice previously, you are already using namespaces.

You can freely add new namespaces: the whole idea is to deploy your Pods and other objects in Kubernetes, making sure to always define a namespace of your choosing. This way, you will maintain a clean and well-organized cluster. We will discover right after that by default, Kubernetes is created with a namespace, and it is the one that is used if you do not specify another explicitly.

A key architecting rule is that an application should not be aware of the Kubernetes namespace that it is running in. This means that you should think of Kubernetes namespaces as objects for cluster administrators, not developers or application users. It shouldn't have any impact on how the application works, and an application should be able to deploy on any namespace. If an app is running on namespace A and is then redeployed on namespace B, it should not have any impact on any of its features.

More generally, Kubernetes namespaces will help you achieve the following as an administrator:

Cluster partitioning to ease resource organization
Scoping resource names
Hardware sharing and consumption limitation
Permissions access
I recommend that you create one namespace per microservice and deploy all the resources that belong to a microservice within its namespace. However, be aware that Kubernetes does not impose any specific rules on you. For example, you could decide to use namespaces in these ways:

Differentiate environments
For example, one namespace for a production environment and another one for a development environment.

Differentiate the tiers
One namespace for databases, one for application Pods, and another for middleware deployment.

Just using the default namespaces
For the smallest clusters that only deploy a few resources, you can go for the simplest setup and just use one big default namespace and deploy everything into it.

Either way, keep in mind that even though two Pods are deployed in different namespaces and exposed through services, they can still interact and communicate with each other. Even though Kubernetes services are created in a given namespace, they'll receive a fully qualified domain name (FQDN) that will be accessible on the whole cluster. So, even if an application running on namespace A needs to interact with an application in namespace B, it will have to call the service exposing app B by its FQDN. You don't need to worry about cross-namespace communication.

**Switching between namespaces with kubectl**

When using kubectl to use Kubernetes, you saw that you have to use the -n or ---namespace option with your kubectl command to tell it the namespace you're targeting. But it is also possible to define a namespace at the configuration level. This way, you'll have kubectl always running requests against that namespace, even if you omit the -n parameter.

Let's create a few more namespaces. Then, we will demonstrate how to switch between them without the -n flag:

```$ kubectl create ns another-ns```

namespace/another-ns created

At the moment, we know that we have three namespaces in our cluster. You can use the following command to switch to another namespace:

```$ kubectl config set-context $(kubectl config current-context) --namespace=another-ns```

Context "minikube" modified.

Running this command will update the current configuration that kubectl is using to make it point to the specified namespace. At the moment, any command you run will be executed against the another-ns namespace.

IMPORTANT NOTE

Running the kubectl config command and sub-commands will only trigger modification or read operations against the ~/.kube/config file, which is the configuration file that kubectl is using.

When you're using the kubectl config set-context command, you're just updating that file to make it point to another namespace.

Knowing how to switch between namespaces with kubectl is important, but before you run any write operations such as kubectl delete or kubectl create, make sure that you are in the correct namespace. Otherwise, you should continue to use the -n flag. As this switching operation might be executed a lot of times, Kubernetes users tend to create Linux aliases to make them easier to use. Do not hesitate to define a Linux alias if you think it can be useful to you.

Understanding why you should set ResourceQuota
Just like applications or systems, Kubernetes Pods will require a certain amount of computing resources to work properly. In Kubernetes, you can configure two types of computing resources:

CPU
Memory
All your worker nodes work together to provide CPU and memory, and in Kubernetes, adding more CPU and memory simply consists of adding more worker nodes to make room for more Pods. Depending on whether your Kubernetes cluster is based on-premises or in the cloud, adding more worker nodes can be achieved by purchasing the hardware and setup to do so or simply calling an API to create additional virtual machines.

Understanding how Pods consume these resources
When you launch a Pod on Kubernetes, a control plane component, known as kube-scheduler, will elect a worker node and assign the Pods to it. Then, the kubelet on the elected worker node will attempt to launch the containers defined in the Pod.

This process of worker node election is called Pod scheduling in Kubernetes.

When a Pod gets scheduled and launched on a worker node, it has, by default, access to all the resources the worker node has. Nothing is preventing it from accessing more and more CPU and memory as the application is used and ultimately, if the Pods run out of memory or CPU resources to work properly, then it simply crashes.

This can become a real problem because worker nodes can be used to run multiple applications – and so multiple Pods – at the same time. So, if 10 Pods are launched on the same worker node but one of them is consuming all the computing resources, then this will have an impact on all 10 Pods running on the worker node.

This problem means that you have two things you must consider:

Each Pod should be able to require some computing resource to work.
The cluster should be able to restrict the Pod's consumption so that it doesn't take all the resources available and share them with other Pods too.
It is possible to address these two problems in Kubernetes and we will discover how to use two options that are exposed to the Pod object. The first one is called resources, which is the option that's used to let a Pod indicate what amount of computing resources it needs, while the other one is called limit and will be used to indicate the maximum computing resources the Pod will have access to.

Let's discover these options now.

Understanding how Pods can require computing resources
The request and limit options will be declared within the YAML definition file of a Pod resource. Here, we're going to focus on the request option.

request is not a resource kind on its own – it is simply an option you can directly specify within your YAML definition file. request is simply the minimal amount of computing resource a Kubernetes Pod will need to work properly, and it is a really good practice to always define a request option for your Pods, at least for those that are meant to run in production.

Let's say that you want to launch an NGINX Pod on your Kubernetes cluster. By filling in the request option, you can tell Kubernetes that your NGINX Pod will need, at the bare minimum, 512 MiB of memory and 25% of a CPU core to work properly.

Here is the YAML definition file that will create this Pod:

# ~/Pod-in-namespace-with-request.yaml

apiVersion: v1

kind: Pod

metadata:

  name: nginx-with-request

  namespace: custom-ns

spec:

  containers:

  - name: nginx

    image: nginx:latest

    resources:

      requests:

        memory: "512Mi"

        cpu: "250m"

As you can see, you can define request at the container level and set different ones for each container.

There are three things to notice about this Pod:

It is created inside the custom-ns namespace.
It requires 512Mi of memory.
It requires 250m of CPU.
But what do these metrics mean?

Memory is expressed in bytes, whereas CPU is expressed in millicores and allows fractional values. If you want your Pod to consume one entire CPU core, you can set the cpu key to 1000m. If you want two cores, you must set it to 2000m; for half of a core it will be 500m or 0.5, and so on. So, to explain, the preceding YAML definition is saying that the NGINX Pod we will create will forcibly need 512 MiB of memory since memory is expressed in bytes, and one-quarter of a CPU core of the underlying worker node. There is nothing related to the CPU or memory frequency here.

When you apply this YAML definition file to your cluster, the scheduler will look for a worker node that is capable of launching your Pods. This means that you need a worker node where there is enough room in terms of available CPU and memory to meet your Pods' requests.

But what if no worker node is capable of fulfilling these requirements? Here, the Pod will never be scheduled and never be launched. Unless you remove some running Pods to make room for this one, or unless you add a worker node that is capable of launching this Pod, it won't ever be launched.

IMPORTANT NOTE

Keep in mind that Pods cannot span multiple nodes. So, if you set 8,000 m, which represents eight CPU cores, but your cluster is made up of two worker nodes with four cores each, then no worker node will be able to fulfill the request and your Pod won't be scheduled. That's why it's not good to set requests with really high values.

So, use the request option wisely – consider it as the minimum amount of resources your Pod will need to work. You have the risk that your Pod will never be scheduled if you set too high a request, but on the other hand, if your Pod is scheduled and launched successfully, this amount of resources is guaranteed.

Understanding how you can limit resource consumption
When you write a YAML definition file, you can define resource limits regarding what the Pod will be able to consume.

Setting a request won't suffice to do things properly. You should set a limit each time you set a resource. Setting a limit will tell Kubernetes to let the Pod consume resources up to that limit, and never above. This way, you're ensuring that your Pod won't take all the resources of the worker for itself.

Be careful, though – Kubernetes won't behave the same, depending on what kind of limit is reached. If the Pod reaches its CPU limit, it is going to be throttled and you'll notice performance degradation. But if your Pod reaches its memory limit, then it might be terminated. The reason for this is that memory is not something that can be throttled and Kubernetes still needs to ensure that other applications are not impacted and remain stable. So, be aware of that.

Without a limit, the Pod will be able to consume all the resources of the worker node. Here is an updated YAML file corresponding to the NGINX Pod we saw earlier, but now, it has been updated to define a limit on memory and CPU.

Here, the Pod will be able to consume up to 1 GiB of memory and up to 1 entire CPU core of the underlying worker node:

# ~/Pod-in-namespace-with-request-and-limit.yaml

apiVersion: v1

kind: Pod

metadata:

  name: nginx-with-request-and-limit

  namespace: custom-ns

spec:

  containers:

  - name: nginx

    imaage: nginx:latest

    resources:

      requests:

        memory: "512Mi"

        cpu: "250m"

      limits:

        memory: "1Gi"

        cpu: "1000m"

So, when you set a request, set a limit too. Now that you are aware of this request and limit consideration, don't forget to add it to your Pods!

Understanding why you need ResourceQuota
The good news is that you can entirely manage your Pod's consumptions by relying entirely on the Pod's request and limits options. All the applications in Kubernetes are just Pods, so setting these two options provides you with a strong and reliable way to manage resource consumption on your cluster, given that you never forget to set these.

It is super easy to forget these two options and deploy a Pod on your cluster that won't define any request or limit. Maybe it will be you, or maybe a member of your team, but the risk of deploying such a Pod is high because everyone can forget about these two options. And if you do so, the risk of application instability is high because a Pod without a limit can eat all the resources on the worker node is it launched on.

Kubernetes provides a way to mitigate this issue thanks to two objects called ResourceQuota and LimitRange. These two objects are extremely useful because they can enforce these constraints at the namespace level.

ResourceQuota is another resource kind, just like Pod or ConfigMap. The workflow is quite simple and consists of two steps:

First, you must create a new namespace.
Then, you must create a ResourceQuota and a LimitRange object inside that namespace.
Then, all the Pods that are launched in that namespace will be constrained by these two objects.

These quotas are used, for example, to ensure that all the containers that are accumulated in a namespace do not consume more than 4 GiB of RAM.

Therefore, it is possible and even recommended to set restrictions on what can and cannot run within Pods. It is strongly recommended that you always define a ResourceQuota and a LimitRange object for each namespace you create in your cluster!

Without these quotas, the deployed resources could consume as much CPU or RAM as they want, which could ultimately make your cluster and all the applications running on it unstable, given that the Pods don't hold requests and limits as part of their respective configuration.

In general, ResourceQuota is used to do the following:

Limit CPU consumption within a namespace
Limit memory consumption within a namespace
Limit the absolute number of Pods operating within a namespace
There are a lot of use cases and you can discover all of them directly in the Kubernetes documentation. Now, let's learn how to define ResourceQuota in a namespace.

Creating a ResourceQuota
To demonstrate the usefulness of ResourceQuota, we are going to create one for a namespace we are going to call custom-ns-with-resource-quota. This ResourceQuota will be used to create requests and limits that all the Pods within this namespace combined will be able to use. Here is the YAML file that will create ResourceQuota; please note the resource kind:

# ~/resourcequota.yaml

apiVersion: v1

kind: ResourceQuota

metadata:

  name: my-resourcequota

spec:

  hard:

    requests.cpu: "1000m"

    requests.memory: "1Gi"

    limits.cpu: "2000m"

    limits.memory "2Gi"

ResourceQuota is enforcing some requests and limits on all the Pods that will be launched in the namespace where it is created. Keep in mind that the ResourceQuota object is scoped to one namespace.

This one is stating that in this namespace, the following will occur:

All the Pods combined won't be able to request more than one CPU core.
All the Pods combined won't be able to request more than 1 GiB of memory.
All the Pods combined won't be able to consume more than two CPU cores.
All the Pods combined won't be able to consume more than 2 GiB of memory.
You can have as many Pods and containers in the namespace, so long as they are respecting these constraints. Most of the time, ResourceQuotas are used to enforce constraints on requests and limits, but they can also be used to enforce these limits per namespace.

In the following example, the preceding ResourceQuota has been updated to specify that the namespace where it is created cannot hold more than 10 ConfigMaps and 5 services, which is pointless but a good example to demonstrate the different possibilities with ResourceQuota:

# ~/resourcequota-with-object-count.yaml

apiVersion: v1

kind: ResourceQuota

metadata:

  name: my-resourcequota

spec:

  hard:

    requests.cpu: "1000m"

    requests.memory: "1Gi"

    limits.cpu: "2000m"

    limits.memory "2Gi"

    configmaps: "10"

    services: "5"

Lastly, when you apply a ResourceQuota YAML definition file, don't forget to add the --namespace setting to set the namespace where ResourceQuota will be applied:

$ kubectl create -f ~/resourcequota-with-object-count.yaml --namespace=custom-ns

Now, let's learn how to list ResourceQuotas.

Listing ResourceQuota
ResourceQuota objects can be accessed through kubectl using the quota's resource name option. The kubectl get command will do this for us:

$ kubectl get quotas -n custom-ns

Now, let's learn how to delete ResourceQuota from a Kubernetes cluster.

Deleting ResourceQuota
To remove a ResourceQuota object from your cluster, use the kubectl delete command:

$ kubectl delete -f quotas/resourcequota-with-object-count -n custom-ns

Now, let's introduce the notion of LimitRanges.

Introducing LimitRange
LimitRange is another object that is similar to ResourceQuota as it is created at the namespace level. The LimitRange object is used to enforce default requests and limit values to individual containers. Even by using the ResourceQuota object, you could create one object that consumes all the available resources in the namespace, so the LimitRange object is here to prevent you from creating too small or too large containers within a namespace.

Here is a YAML file that will create LimitRange:

# ~/limitrange.yaml

apiVersion: v1

kind: LimitRange

metadata:

  name: my-limitrange

spec:

  limits:

  - default:

      memory: 256Mi

      cpu: 500m

    defaultRequest:

      memory: 128Mi

      cpu: 250Mib

    max:

      memory: 1000Mi

      cpu: 1Gib

    min:

      memory: 128Mi

      cpu: 250m

    type: Container

As you can see, the LimitRange object consists of four important keys that all contain memory and cpu configuration. These keys are as follows:

default: This helps you enforce default values for the memory and cpu limits of containers if you forget to apply them at the Pod level. Each container that is set up without limits will inherit these default ones from the LimitRange object.
defaultRequest: This is the same as default but for the request option. If you don't set a request option to one of your containers in a Pod, the ones from this key in the LimitRange object will be automatically used by default.
max: This value indicates the maximum limit (not request) a Pod can set. You cannot configure a Pod with a limit value that is higher than this one. It is the same as the default value in that it cannot be higher than the one defined here.
min: This value works like max but for requests. It is the minimum amount of computing resources a Pod can request, and the defaultRequest option cannot be lower than this one.
Finally, note that if you omit the default and defaultRequest keys, then the max key will be used as the default key, and the min key will be used as the default key.

Defining LimitRange is a good idea if you want to protect yourself from forgetting to set requests and limits on your Pods. At least with LimitRange, these objects will have default limits and requests!

Just like the ResourceQuota object, don't forget to set the -n option to create LimitRange inside a namespace:

$ kubectl create -f ~/limitrange.yaml --namespace=custom-ns

Now, let's learn how to list LimitRanges.

Listing LimitRange
The kubectl command line will help you list your LimitRanges. Don't forget to add the -n flag to scope your request to a specific namespace:

$ kubectl get limit -n custom-ns

Now, let's learn how to delete LimitRange from a namespace.

Deleting LimitRange
Deleting LimitRange can be achieved using the kubectl command-line tool. Here is how to proceed:

$ kubectl delete limit/my-limitrange -n custom-ns

As always, don't forget to add the -n flag to scope your request to a specific namespace; otherwise, you may target the wrong one!