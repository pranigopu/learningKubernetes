# Kubernetes architecture
## Basic terms
_Note that all these terms are defined in the context of Kubernetes._

> Reference for the following (search as needed): https://kubernetes.io/docs/concepts/

**NOTE**: Kubernetes object means Kubernetes API-object (i.e. an object from the perspective of the Kubernetes API).

### Pod
A pod is a logical wrapper within Kubernetes for one or more containers. **_In essence, a pod represents a set of running containers on your cluster_**. Note that just as a container is the basic software unit of Docker, so is pod the basic software unit (_i.e. the fundamental building block_) of Kubernetes. Containers are encapsulated within pods. Some points on pods...

- A pod is the smallest object creatable in Kubernetes
- Pods usually have 1-1 relationship with containers
	- Pods _may_ have multiple containers (rare cases)
	- There cannot be multiple identical containers (i.e. replicas of containers) in one pod
- Scaling with pods
	- To scale up: create replicas of a pod or pods
	- To scale down: deleting replicas of a pod or pods
	- Do not scale within pods (i.e. do not add replicas of containers within pods to scale up)

**NOTE 1**: Since pods are intended to be disposable and replaceable, you cannot add a container to a pod once it has been created. Instead, you usually delete and replace pods using deployments.

**NOTE 2**: Pods in Kubernetes are ephemeral in nature and do not persist data, so the data in a pod is lost once it is destroyed or restarted (which also happens when you restart the cluster in AKS). To support persistent data storage, Kubernetes provides a PersistentVolume (PV) and a PersistentVolumeClaim (PVC) object.

> References for notes 1 and 2:
> - https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers
> - https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7865615

### Node, cluster & node pool
A **node** is a machine (physical or virtual) that provides the platform for pods (and thus containers and containerized applications) to run on. A **cluster** is a collection of nodes managed by a control plane (discussed further). The collection of nodes itself is called a **node pool** and becomes a cluster when combined with a control plane.

### ReplicaSet
A ReplicaSet (RS) is a resource object that acquires and maintains (by creating or deleting) a specified number of replicas of specific container(s) within a node or cluster. It is used to guarantee the availability of a specified number of identical containers (hence, identical instances of the application(s) running on the containers).

The purpose of a ReplicaSet is to maintain a stable set of running replicas of a container (i.e. running instances of an application) at any given time. _Note that each replica of a container is also a container itself, and like all containers in Kubernetes, each replica is wrapped in a replicated pod_. In practice, we provide a container image for the ReplicaSet to replicate; the container is created and replicated by the ReplicaSet according to the way we define the ReplicaSet.

ReplicaSets do the following:

- Ensure high availability and reliability of applications (_by creating and maintaining replicas of certain container(s), which serve as backups in case of failure_)
- Enable seamless scaling (_by creating or deleting replicas as needed_)
- Load balancing (_by distributing requests appropriately to the replicas_ **_using services that are a part of the ReplicaSet_**)

### Controller
A controller is a software component that runs control processes. A controller process is a looping process that watches the state of the cluster (through the Kubernetes API server _which discussed later in this document_) and makes changes to try to push the cluster to the desired state (the current and desired state are evaluated based on some metric or set of metrics, such as node availability, job-completion, etc.).

### Deployment
A deployment is a resource object that provides declarative updates to applications (i.e. to containerized applications running on pods or ReplicaSets). To elaborate, we can describe a desired state (i.e. a desired combination of properties for the application(s) being deployed) and the **deployment controller** changes the actual state of the targeted application(s) and associated environment to the desired state _at a controlled rate_.

**NOTE 1**: Any changes being applied to the deployment will be immediately applies to the application. Such a change generally involves the creation of new pods or ReplicaSets and the termination of old pods or ReplicaSets, in order to align with the deployment's new specifications.

**NOTE 2**: As a deployment is updated, it maintains a number of its older versions (usually 10 older versions) to which we can roll back when we want.

We can do the following with deployments:

- Roll out a ReplicaSet through a deployment (_happens by default when creating a deployment_)
- Update a deployment (_i.e. update applications and their environment_)
- Roll back to previous versions of a deployment (_evey rollback is counted as an update, so no "roll forwards" are possible_)
- Scale a deployment (_similar to scaling a ReplicaSet_)
- Pausing and resuming a deployment (_to make changes without simultaneously creating and destroying pods_)
- Observe the deployment status of the application(s) being deployed
- Canary deployments (_i.e. distributing traffic to new and old versions of the deployment, in order to test the newer version_)

> Extra reference: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

### Policy
Kubernetes policies are configurations (_i.e. settings or specifications_) that manage runtime behaviors or other configurations.

### Service
An abstraction that defines a logical set of pods and a policy by which to access them. The particular pods in the set are disposable and replaceable (_as long as their replacements can satisfy the required role_).

A service is a method for exposing a network application (_i.e. an application running on a network of interconnected nodes_) that is running as one or more pods in the cluster. A service also provides a stable virtual ID by which to access the network application despite it being run on a set of disposable pods that may vary over time. Lastly, a service also acts as a load balancer for the set of pods associated to it at a given point in time. Note that a service may either expose a set of pods internally (within the cluster's virtual network) or externally (outside the cluster's virtual network). An exception is the _ExternalName_ service (discussed more shortly), which exposes an external resource to the cluster rather than exposing the cluster's pods.

Services and pods are connected through labels and selectors, wherein the service is given a selector field to help select the pods with the given labels. Of course, an _ExternalName_ service works differently, using the cluster's DNS to look for, map to and expose the particular external resource.

**NOTE**: Just as container instances within pods can be accessed using the `kubectl run` command, so can a service itself (_wherein we would be accessing the networked application being exposed by the service, which could as well be a single container instance_).

> Extra references:
> - On services in general: https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/\
> - On _ExternalName_ service: https://www.ithands-on.com/2021/04/kubernetes-101-external-services.html

#### Service types
To expose pods internally (i.e. within the cluster), we use the _ClusterIP_ service. To expose pods externally (i.e. both within and outside the cluster), we can use _LoadBalancer_, _NodePort_, _Ingress_ or _ExternalName_ services. Note that the _ClusterIP_ service is the default; without specifying the service type, a created service defaults to _ClusterIP_.

##### ExternalName service
This service maps itself to the contents of the  `externalName`  field in the YAML manifest (for example, to the hostname  `api.x.x.x`). In other words, it maps to and thus exposes an external resource (such as a database server) to the objects within the cluster. More specifically, the mapping configures your cluster's DNS server to return a  `CNAME`  record (_CNAME means "Canonical Name", i.e. the true name of the resource, as opposed to an alias_) with that external hostname value (i.e. `api.x.x.x`).

**NOTE**: The hostname value becomes to alias, whereas the IP address of the resource is the CNAME.

> Extra references:
> - https://kubernetes.io/docs/concepts/services-networking/service/#externalname
> - https://www.ithands-on.com/2021/04/kubernetes-101-external-services.html

#### Azure Kubernetes Service (AKS)
When creating a Kubernetes cluster in an Azure resource group, Azure creates an Azure Kubernetes Service (AKS) that exposes the pods of the whole cluster to our Azure subscription. In essence, an AKS cluster is in fact a service that exposes the whole cloud-native Kubernetes cluster (the Kubernetes cluster that resides in Azure's virtual machines) to our Azure subscription so that we (the Azure subscriber) can access and manage the cloud-native Kubernetes cluster.

To us, the users, AKS is presented as a ClusterIP service that exposes the cluster internally, but in fact, AKS is exposing the actual Kubernetes cluster externally, i.e. to the cloud provider subscription.

#### Frontend & backend
With respect to exposing application through a service, frontend refers to the software component that is connected to the broader network as well as the cluster, while maintaining a mapping between the IP addresses of the cluster and their virtual network addresses (i.e. internal addresses within the cluster). The backend refers to the components on which the application is running (the pods, nodes, etc.).

### Workload
An application running on Kubernetes. Regardless of whether a workload has a single component or several, it is run within a set of pods.

### Service account
A type of non-human account (i.e. unique identification) that provides a distinct identity in a cluster. A service account is managed by a _ServiceAccount_ object in Kubernetes. Pods, nodes and other entities inside and outside the cluster can use a specific _ServiceAccount_'s credentials to identify as that _ServiceAccount_. This identity is useful in various situations, including authenticating to the API server or implementing identity-based security policies.

---

_We have only looked at the concepts needed to understand the Kubernetes architecture. More Kubernetes-related concepts are discussed in another document_.

## Kubernetes architecture
### Container runtime
_Kubernetes does not actually handle the process of running containers on a machine. Instead, it relies on another piece of software called a container runtime._

A container runtime is the software that runs containers on a host. More precisely, it is a low-level component of a container engine that works with the OS kernel of the host to start and support the containerization process as well as mount containers.

**NOTE 1**: A container runtime is only the low-level component of the container engine, not the engine itself; the container runtime implements the instructions passed down from the higher levels of the engine.

**NOTE 2**: Since a container runtime runs for given host, **_a container runtime must be installed for each node in a Kubernetes cluster_**.

The Kubernetes engine runs on top of a container runtime. Common container runtimes used by Kubernetes are:

- containerd
- Docker Engine
- CRI-O
- Mirantis Container Runtime

> Extra reference: https://medium.com/tektutor/container-engine-vs-container-runtime-667a99042f3

### Control plane & node pool
Pods are hosted on nodes, and nodes are managed as clusters. The collection of nodes itself is called the node pool, and the software system that manages the nodes and pods in the node pool is called the control plane. The node pool and control plane together form a cluster.

The control plane itself has one or more nodes to run services required to manage and control the cluster. Hence, we get the following differentiation of nodes in a cluster:

- **Master node(s)**: nodes of the control plane that run services required to manage and control the cluster
- **Worker node(s)**: nodes of the node pool that host pods (i.e. that run containerized applications)

**SIDE NOTE**: Note that while one master node is often sufficient at any given point in time, the control plane in production environments usually runs across multiple master nodes (i.e. multiple computer systems), providing fault-tolerance and high availability.

### 1. Control plane components

> Reference for the following: https://kubernetes.io/docs/concepts/overview/components/

#### Overview
Control plane components (which run on master nodes) manage and control the cluster by:

1. Making global decisions for the cluster (_ex. scheduling_)
2. Detecting and responding to cluster events (_ex. starting a new pod when there are insufficient replicas of that kind of pod_)

Control plane components can be run on any machine or set of machines in the cluster. However, for simplicity, control plane components are started on the same machine, and this machine does not run user containers.

#### 1.1. kube-apiserver
A Kubernetes API server exposes the Kubernetes API to other control plane components, i.e. it is the Kubernetes API's frontend for the control plane. To elaborate, a Kubernetes API server is the core of a Kubernetes cluster's control plane, since this server exposes an HTTP API that lets (1) end users, (2) different parts of your cluster, and (3) external components communicate with one another.

**NOTE**: _The Kubernetes API is an HTTP API that lets you query and manipulate the state of objects in Kubernetes clusters (ex. pods, namespaces, configuration maps (ConfigMaps), services and events)_.

The main implementation of a Kubernetes API server is kube-apiserver. kube-apiserver scales horizontally, i.e. it deploys more instances of itself to scale up. Several instances of kube-apiserver can be run such that traffic to and from the control plane is balanced among these instances.

> Extra reference: https://kubernetes.io/docs/concepts/overview/kubernetes-api/

#### 1.2. etcd
etcd is a consistent and highly available key-value store (i.e. a storage that stores associated key and value pairs, i.e. one key associated with one value). It is used as a backing store (i.e. secondary or persistent storage) by a Kubernetes cluster to store the cluster's data.

**NOTE 1**: Consistency in memory means its contents always reflect the intended result of read and write operations. This may mean maintaining mutual exclusivity among write operations and allowing reads only when no writes are being done.

**NOTE 2**: High availability in a system (ex. memory) means it always stays online, i.e. it always stays available -- even if partly -- regardless of failure.

#### 1.3. kube-scheduler
kube-scheduler watches for newly created pods and -- if they are not assigned to a node -- assigns them to selected nodes for them to run on. Such an assignment is called a "scheduling decision".

Scheduling decisions may be done based on various factors, such as:

- **Resource requirements**
	- Individual
	- Collective (of all available pods)
- **Constraints**
	- Hardware
	- Software
	- Policy
- **Data locality** (_the process of moving computation to a node to which the required data is closer to access -- ideally, locally available in the node_)
	- _Reduces congestion due to reduced need to move data_
	- _Faster operation for the same reason_
- **Deadlines for completion of tasks**
- **Minimizing interference between workloads** (_i.e. applications running on the pods_)

#### kube-controller-manager
The component that runs the essential controller processes in the Kubernetes cluster. _To recap from a previous section_, a controller process is a looping process that watches the state of the cluster (through the API server) and makes changes to try to push the cluster to the desired state (the current and desired state are evaluated based on some metric or set of metrics, such as node availability, job-completion, etc.).

Logically (_i.e. by conceptual design_), each controller process is a separate process. However, for the sake of reduced space and time complexity, controller processes are compiled into a single binary (_i.e. a single executable program_) and run as a single process (_i.e. several independent controller processes are combined into one_).

Some type of controller processes (also called simply "controllers"):

- **Node controller**
	- Notices and responds when nodes go down
- **Job controller**
	- Watches for "Job" objects representing one-off tasks and creates pods to run these tasks to completion
- **Endpoint controller**
	- Provides and manages links between services and pods
- **Service account controller**
	- Creates default _ServiceAccount_ objects for new namespaces.
	
#### cloud-controller-manager
The component that runs cloud-specific controller processes, i.e. controller processes specific to the cloud platform being used (if used). On-premise Kubernetes clusters (_which do not use a cloud platform_) do not have this component.

The cloud controller manager...

- Enables the cluster to be linked to the cloud provider's API
- Only runs controller processes specific to the cloud provider

_As with kube-controller-manager_, logically (_i.e. by conceptual design_), each controller process is a separate process. However, for the sake of reduced space and time complexity, controller processes are compiled into a single binary (_i.e. a single executable program_) and run as a single process (_i.e. several independent controller processes are combined into one_).

Some type of cloud controller processes:

-  **Node controller**
	- Checks if an unresponsive node is deleted in cloud or not
- **Route controller**
	- For setting up routes between the cluster components and nodes (which may both be cloud-native) in the underlying cloud infrastructure
- **Service controller**
	- For creating, updating and deleting the load balancers of the cloud provider

### 2. Node components

> Reference for the following: https://kubernetes.io/docs/concepts/overview/components/

#### 2.1. kubelet
A software agent running on each node in the cluster to ensure that containers (_created by Kubernetes_) run in pods. It also ensures that the containers within pods are running and healthy (i.e. working as intended). Note that a kubelet does not manage containers not created by Kubernetes.

### 2.2. kube-proxy
A network proxy running on each node in the cluster to maintain network rules on nodes (_network rules allow network communication to the pods from network sessions (i.e. established, time-bound connections) inside or outside of your cluster_). As a network proxy, kube-proxy also plays a role in implementing the concept of Kubernetes services.

kube-proxy forwards traffic to and from the node either (1) by itself or (2) using an OS packet filtering layer (if available).
