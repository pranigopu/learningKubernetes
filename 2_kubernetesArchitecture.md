# Kubernetes architecture
## Basic terms
_Note that all these terms are defined in the context of Kubernetes._

> Reference for the following (search as needed): https://kubernetes.io/docs/concepts/

### Pod
A pod is a logical wrapper within  Kubernetes for one or more containers. **_In essence, a pod represents a set of running containers on your cluster_**. Note that just as a container is the basic software unit of Docker, so is pod the basic software unit of Kubernetes. Containers are encapsulated within pods. Some points on pods...

- A pod is the smallest object creatable in Kubernetes
- Pods usually have 1-1 relationship with containers
	- Pods _may_ have multiple containers (rare cases)
	- There cannot be multiple identical containers (i.e. replicas of containers) in one pod
- Scaling with pods
	- To scale up: create replicas of a pod or pods
	- To scale down: deleting replicas of a pod or pods
	- Do not scale within pods (i.e. do not add replicas of containers within pods to scale up)

### Node, cluster & node pool
A **node** is a machine (physical or virtual) that provides the platform for pods (and thus containers and containerized applications) to run on. A **cluster** is a collection of nodes managed by a control plane (discussed further). The collection of nodes itself is called a **node pool** and becomes a cluster when combined with a control plane.

### ReplicaSet
A ReplicaSet (RS) is a Kubernetes object that acquires and maintains (by creating or deleting) a specified number of replicas of specific kind of pod within a node or cluster. It is used to guarantee the availability of a specified number of identical pods.

### Policy
Kubernetes policies are configurations (_i.e. settings or specifications_) that manage runtime behaviors or other configurations.

### Service
An abstraction that defines a logical set of pods and a policy by which to access them. A logical set of pods is a set that specifies a certain number of pods of a certain kind without specifying the particular pods to be used; the particular pods are interchangeable as long as they can satisfy their required role.

A service is a method for exposing a network application (_i.e. an application running on a network of interconnected nodes_) that is running as one or more pods in the cluster. A service also provides a stable virtual ID by which to access the network application despite it being run on a set of disposable pods that may vary over time. Lastly, a service also acts as a load balancer for the set of pods associated to it at a given point in time.

> Extra reference: https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/

### Workload
An application running on Kubernetes. Regardless of whether a workload has a single component or several, it is run within a set of pods.

### Service account
A type of non-human account (i.e. unique identification) that provides a distinct identity in a cluster. A service account is managed by a _ServiceAccount_ object in Kubernetes. Pods, nodes and other entities inside and outside the cluster can use a specific _ServiceAccount_'s credentials to identify as that _ServiceAccount_. This identity is useful in various situations, including authenticating to the API server or implementing identity-based security policies.

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
A Kubernetes API server exposes the Kubernetes API to other control plane components, i.e. it is the Kubernetes API's frontend for the control plane.

The main implementation of a Kubernetes API server is kube-apiserver. kube-apiserver scales horizontally, i.e. it deploys more instances of itself to scale up. Several instances of kube-apiserver can be run such that traffic to and from the control plane is balanced among these instances.

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
The component that runs controller processes. A controller process is a looping process that watches the state of the cluster (through the API server) and makes changes to try to push the cluster to the desired state (the current and desired state are evaluated based on some metric or set of metrics, such as node availability, job-completion, etc.).

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