**Why you would want to use PersistentVolume**

When you're creating your Pods, you have the opportunity to create volumes in order to share files between the containers created by them. However, these volumes can represent a massive problem: they are bound to the life cycle of the Pod that created them.

That is why Kubernetes offers another object called PersistentVolume, which is a way to create storage in Kubernetes that will not be bound to the life cycle of a Pod.

Additionally, we discovered the emptyDir and hostPath volumes, which, respectively, initiate an empty directory that your Pods can share. It also defines a path within the worker Node filesystem that will be exposed to your Pods. Both of these volumes were supposed to be attached to the life cycle of the Pod. This means that once the Pod is destroyed, the data stored within the volumes will be destroyed as well.

However, sometimes, you don't want the volume to be destroyed. You just want it to have its life cycle to keep both the volume and its data alive even if the Pod fails. That's where PersistentVolumes comes into play: essentially, they are volumes that are not tied to the life cycle of a Pod. Since they are a resource kind just like the Pods themselves, they can live on their own!

__IMPORTANT NOTE__

Bear in mind that PersistentVolumes objects are just entries within the etcd datastore, and they are not an actual disk on their own.

PersistentVolume is just a kind of pointer within Kubernetes to an external piece of storage, such as an NFS, a disk, an Amazon EBS volume, and more. This is so that you can access these technologies from within Kubernetes and in a Kubernetes way.

Simply put, PersistentVolume is essentially made up of two different things:

A backend technology called a PersistentVolume type
An access mode, such as ReadWriteOnce
You need to master both concepts in order to understand how to use PersistentVolumes. Let's begin by explaining what PersistentVolume types are.

__Introducing PersistentVolume types__

Kubernetes is supposed to be able to run on as much infrastructure as possible, and even though it started as a Google project, it can be used on many platforms, whether they are public clouds or private solutions.

As you already know, the simplest Kubernetes setup consists of a simple minikube installation, whereas the most complex Kubernetes setup can be made of dozens of servers on massively scalable infrastructure. All of these different setups will forcibly have different ways in which to manage persistent storage. For example, the three biggest public cloud providers have a lot of different solutions. Let's name a few, as follows:

Amazon AWS EBS volumes
Amazon AWS EFS filesystems
Google GCE Persistent Disk (PD)
Microsoft Azure disks
These solutions have their own design and set of principles along with their own logic and mechanics. Kubernetes was built with the principle that all of these setups should be abstracted using just one object to abstract all of the different technologies. And that single object is the PersistentVolume resource kind. The PersistentVolume resource kind is the object that is going to be attached to a running Pod. Indeed, a Pod is a Kubernetes resource and does not know what an EBS or a PD is; Kubernetes Pods only play well with PersistentVolumes, which is also a Kubernetes resource.

Whether your Kubernetes cluster is running on Google GKE, Amazon EKS, or whether it is a single Minkube cluster on your local machine has no importance. When you wish to manage persistent storage, you are going to create, use, and deploy PersistentVolumes objects, and then bind them to your Pods!

Here are some of the backend technologies supported by Kubernetes out of the box:

awsElasticBlockStore: Amazon EBS volumes
gcePersistentDisk: Google Cloud PD
azureDisk: Azure Disk
azureFile: Azure File
cephfs: Ceph-based filesystems
csi: Container storage interface
glusterfs: GlusterFS-based filesystems
nfs: Regular network file storage
The preceding list is not exhaustive: Kubernetes is extremely versatile and can be used with many storage solutions that can be abstracted as PersistentVolume objects in your cluster.

When you create a PersistentVolume object, essentially, you are creating a YAML file. However, this YAML file is going to have a different key/value configuration based on the backend technology used by the PersistentVolume objects.

The benefits brought by PersistentVolume
There are three major benefits of PersistentVolume:

PersistentVolume is not bound to the life cycle of a Pod. If you want to remove a Pod that is attached to a PersistentVolume object, then the volume will survive.
The preceding statement is also valid when a Pod crashes: the PersistentVolume object will survive the fault and not be removed from the cluster.
PersistentVolume is cluster-wide; this means that it can be attached to any Pod running on any Node.
Bear in mind that these three statements are not always 100% valid. Indeed, sometimes, a PersistentVolume object can be affected by its underlying technology.

To demonstrate this, let's consider a PersistentVolume object that is, for example, a pointer to an Amazon EBS volume created on your AWS cloud. In this case, the worker Node will be an Amazon EC2 instance. In such a setup, PersistentVolume won't be available to any Node.

The reason is that AWS has some limitations around EBS volumes, which relates to the fact that an EBS volume can only be attached to one instance at a time, and that instance must be provisioned in the same availability zones as the EBS volume. From a Kubernetes perspective, this would only make PersistentVolume (EBS volumes) accessible from EC2 instances (that is, worker Nodes) in the same AWS availability zone, and several Pods running on different Nodes (EC2 instances) won't be able to access the PersistentVolume object at the same time.

However, if you take another example, such as an NFS setup, it wouldn't be the same. Indeed, you can access an NFS from multiple machines at once; therefore, a PersistentVolume object that is backed by an NFS would be accessible from several different Pods running on different Nodes without much problem. To understand how to make a PersistentVolume object on several different Nodes at a time, we need to consider the concept of access modes.

***Introducing access modes***

As the name suggests, access modes are an option you can set when you create a PersistentVolume type that will tell Kubernetes how the volume should be mounted.

PersistentVolumes supports three access modes, as follows:

ReadWriteOnce: This volume allows read/write by only one Node at the same time.
ReadOnlyMany: This volume allows read-only mode by many Nodes at the same time.
ReadWriteMany: This volume allows read/write by multiple Nodes at the same time.
It is necessary to set at least one access mode to a PersistentVolume type, even if said volume supports multiple access modes. Indeed, not all PersistentVolume types will support all access modes.