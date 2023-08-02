# More Kubernetes concepts
_Note that all these terms are defined in the context of Kubernetes._

> Reference for the following (search as needed): https://kubernetes.io/docs/concepts/

**NOTE**: Kubernetes object means Kubernetes API-object (i.e. an object from the perspective of the Kubernetes API).

## ConfigMaps

> Reference: https://kubernetes.io/docs/concepts/configuration/configmap/

### Definition
A ConfigMap is a Kubernetes object used to store non-confidential data in key-value pairs (as in a dictionary).  Pods can use ConfigMaps as environment variables, command-line arguments, or as configuration files for a volume (discussed in the section on storage).

### Purpose
A ConfigMap allows us to decouple environment-specific configuration from our container images, so that our applications (as deployed through Kubernetes) are easily portable.

## Storage in Kubernetes
### 1. Volume
#### 1.1. Introduction to the need
The storage available on containers has the following issues...

**Issue 1**:<br>
On-disk files in a container are ephemeral; container state is not saved, so all of the files that were created or modified during the lifetime of the container are lost. Even during a crash, kubelet restarts the container with a clean memory.

**Issue 2**:<br>
It is difficult to set up a shared filesystem between multiple containers within a pod (_in case the pod has multiple containers_) so that the containers can share files with each other.

Volumes are an abstraction of storage that offer a solution for the above issues.

#### 1.2. Definition
Fundamentally, volume in Kubernetes is a directory (possibly also storing some data) which is accessible to every container in a pod. A volume may be ephemeral (i.e. destroyed when the pod ceases to exist) or persistent (i.e. maintained even after the pod ceases to exist. In any cases, volumes preserve data across container restarts within the pod.

There are different types of volumes in Kubernetes and the type defines:

- How this directory is created
- The medium that backs the directory's contents
- The directory's contents

**NOTE 1**: A pod can use any number of volume types simultaneously.

**NOTE 2**: Volumes are an abstraction of a filesystem containing the data of the containers in a certain hierarchy. How exactly such a storage is achieved depends on the implementation.

> References:
> - https://kubernetes.io/docs/concepts/storage/volumes/
> - https://www.tutorialspoint.com/kubernetes/kubernetes_volumes.htm

### 2. Storage class

> Key references for the following section:
> - https://kubernetes.io/docs/concepts/storage/storage-classes/
> - https://bluexp.netapp.com/blog/cvo-blg-kubernetes-storageclass-concepts-and-common-operations
> - https://github.com/kubernetes/community/blob/master/sig-storage/volume-plugin-faq.md

#### 2.1. Definition
A Kubernetes storage class is a Kubernetes storage mechanism that lets you dynamically provision (i.e. create and allocate) persistent volumes in a Kubernetes cluster. Kubernetes administrators define classes of storage, and then pods can dynamically request the specific type of storage they need. Storage classes can define properties of storage systems such as:

-   Speed (for example, SSD vs. HDD storage)
-   Quality of service levels
-   Backup or replication policies
-   Type of file system
-   Any other property defined by the administrator

**SIDE NOTE**: _Storage classes may also be called "profiles" in other storage systems_.

A Kubernetes cluster typically contains at least one default storage class (cloud providers offering a Kubernetes service (ex. Azure through AKS) may have their own default storage class).

#### 2.2. Provisioner
This property determines how the persistent volumes are provisioned (i.e. using what storage interface). In particular, it defines what volume plugin is used for provisioning persistent volumes. Given below are some key concepts needed to fully grasp this property...

##### Volume plugin
A **volume plugin** is a program that extends the Kubernetes storage interface to support block storage, file storage or both. In fact, since any volume resource in Kubernetes (including persistent volumes) are abstractions of storage, volume and persistent volume are volume plugins in terms of their implementation in Kubernetes.

##### Volume vs. PersistentVolume
Speaking of them as Kubernetes resources (abstracted as objects), Volumes are ephemeral (with respect to the pods to which they were provisioned) while PersistentVolumes have a lifecycle independent of the pods they were provisioned to. Note that PersistentVolumes can be provisioned either by the administrator or dynamically using storage classes.

> Reference: https://kubernetes.io/docs/concepts/storage/persistent-volumes/

##### Internal vs. external provisioners
There are several internal provisioners (i.e. Kubernetes-provided) as well as external provisioners (i.e. independent programs adhering to Kubernetes standards). Note that an external provisioner's creator has complete control over the code's storage location, the provisioner's shipping and operation, and the choice of volume plugins.

> Reference: https://www.knowledgehut.com/blog/devops/kubernetes-storage-class

#### 2.3. Reclaim policy
The **reclaim policy** for a PersistentVolume tells the cluster what to do with the volume after it has been released of its claim. Currently, volumes can either be retained, recycled, or deleted.

Persistent volumes that are dynamically created by a storage class have the reclaim policy specified in the  `reclaimPolicy`  field of the storage class, which can be either  `Delete`  or  `Retain`. If no  `reclaimPolicy`  is specified when a StorageClass object is created, it will default to  `Delete`. PersistentVolumes that are created manually and managed via a StorageClass will have whatever reclaim policy they were assigned at creation

> Extra reference: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaiming

#### 2.4. Other properties
Other properties defined in a storage class are:

- Allowing volume expansion
	- Determines whether the provisioned volumes can be increased in size
	- Volume expansion feature only allows for volume growth not shrinking
	- Default value: `true`
- Mount options
- Volume binding options
	- Determines when persistent volumes should be provisioned
	- Default value: `Immediate` (_i.e. immediately after persistent volume claim is made_)
	- Other values: `WaitForFirstConsumer` (_i.e. only after a pod using the claim is created_)

**NOTE (volume binding options)**: The `WaitForFirstConsumer` option has the advantage that the allocated storage would be placed in the same physical location as the first pod requesting it, which can reduce memory access latency.

### 3. Persistent volume claims (PVCs)

> Reference: https://bobcares.com/blog/kubernetes-persistent-volume-claim

#### 3.1. Introduction to the need
We had mentioned before that volumes, and specifically persistent volumes, can be dynamically allocated to pods using storage classes. The mechanism through which pods request storage via storage classes is PVC.

#### 3.2. Definition
A persistent volume claim (represented by a PersistentVolumeClaim object in Kubernetes) is a persistent storage request that refers to a specific storage class instead of the persistent volume. _Note again that storage classes indicate properties of storage devices such as performance, service levels, and back-end policies, can be defined by administrators; the required amount storage devices matching these properties are assigned to the pods via the storage class_.

**NOTE 1**: The PVC approach of requesting storage in Kubernetes has the advantage of allowing developers to dynamically request storage resources without knowing how the underlying storage devices are implemented.

### 4. Using ConfigMaps to configure volumes in volume requests
More details will be discussed in the documents discussing the practical side. However, note that ConfigMaps can be used to specify the behavior of a containerized application, including say a MySQL application.
