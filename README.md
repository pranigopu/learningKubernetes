
# Learning Kubernetes
## Resources
- https://kubernetes.io/docs/home/

## Introduction
Kubernetes is a portable, extensible open-source container orchestration system for automating software deployment, scaling, and management. In other words, Kubernetes is an open-source orchestration engine for automating deployment, scaling, and management of *containerized applications* i.e. applications packaged into containers. This open-source project is hosted by the *Cloud Native Computing Foundation*.

(**SIDE NOTE**: *"Kubernetes" is often abbreviated as "K8s"* (*i.e. K + 8 letters + s*))

The name Kubernetes originates from Greek, meaning "helmsman" or "pilot". Kubernetes is a software platform that can be installed in any computer system. In practice, Kubernetes runs on top of a container deployment platform such as Docker and automates this platform's operations.

## Containers
Containers are standardized, executable components that combine application source code with the operating system (OS) libraries and dependencies required to run that code in any environment. They virtualise the software layers above the OS level. In other words, a container is an isolated environment for your code, meaning a container has no knowledge of your machine's OS, or your files.

Let us compare and contrast containerised app deployment with other app deployment methods...

### Traditional app deployment

- Applications run on physical servers as they would on a physical computer system
- In the case of multiple applications required by an organisation, we have the following choices:
	- Multiple applications run on one physical server, often leading to resource allocation, process scheduling and performance issues
	- Multiple applications run on one physical server each, often leading to resource wastage

### Virtualised deployment

- Virtual machines are created and run on top of physical servers, with possibly multiple VMs running on one physical server's CPU
- Logically, applications are run on mutually isolated VMs
- Benefits
	- Better resource utilisation
	- Higher security between applications
	- Scalability, since VMs provide an abstraction from the underlying systems, and hence, present physical resources as a set of disposable VMs

### Containerised app deployment

- Similar to a VM, a container has its own filesystem, share of the physical server's CPU, memory, process space, etc.
- Unlike a VM, a container does not virtualise the entire machine (from the hardware layer to the application layer), but rather, only it only virtualises the software layers running on top of the OS (i.e. above the OS level)
- Hence, containers can share an OS, and thus, have a more relaxed isolation than VMs
- Furthermore, since containers are not tied to any particular underlying infrastructure (hardware or OS-based), they are portable across systems and OS distributions

#### In summary...
A container is virtualisation technology similar to a VM, but instead of virtualising the entire machine, it only virtualises the software layers above the OS level. In this way, it packages applications decoupled from any underlying infrastructure, enabling:

- Portability across systems and OS distributions
- Environmental consistency across development, testing, and production, as a containerised application runs the same on a laptop as it does in the cloud
- Resource isolation (similar to VMs), leading to predictable application performance
-   Resource utilization (similar to VMs), leading to high efficiency in terms of memory and CPU usage

Furthermore, it is lightweight as well as easier and more efficient to create and use container images compared to VM images, enabling a more agile development process.

## Need for container orchestration
In a production environment, you need to manage the containers that run the applications and ensure that there is no downtime. This can involve:

- Service discovery
	- Exposing a container for access
- Load balancing
	- If traffic to a container is high, we must be able to distribute the network traffic across multiple containers (offering the same required functionality)
- Storage management within and between containerised applications
- Self-healing
	- This involves restarting containers that fail, replacing containers, killing containers that do not respond to your user-defined health check and not advertising killed containers to clients until they are ready to serve
- Managing configuration of applications
- Managing sensitive information
	- Storing and managing "secrets", i.e. sensitive information such as passwords, authorisation tokens, and SSH keys

As a container orchestration platform, Kubernetes offers the above services and more.

## Basic concepts
### Pod
A pod is a logical wrapper within  Kubernetes for one or more containers. Just as a container is the basic software unit of Docker, so is pod the basic software unit of Kubernetes. Containers are encapsulated within pods.

- Smallest object creatable in Kubernetes
- Pod: single instance of an app
- Usually have 1-1 relationship with containers
	- Pods may have multiple containers (rare cases)
	- Cannot have multiple containers of same kind in one pod
- Scaling
	- Scale up: creating identical pods
	- Scale down: deleting identical pods
	- No scaling within pods, only scaling with pods

### ReplicaSet
### Deployment
### Service
An abstraction for pod, sitting on top of pods and acting as a load balancer while keeping a stable virtual ID.

### Imperative vs. declarative configuring/deploying applications in Kubernetes

## Approach
Due to the complexity of installing, configuring and managing Kubernetes on a personal computer system, we shall be using Azure as the cloud computing platform to access Kubernetes servies. To do this, we shall use AKS (Azure Kubernetes Service), which offers the quickest way to start developing and deploying cloud-native apps (i.e. apps that are stored and run on the cloud) in Azure with built-in code-to-cloud pipelines and guardrails. As a Kubernetes service hosted on the Azure platform, AKS handles critical tasks, like health monitoring and maintenance, and is highly available, secure and fully managed.

## Kubernetes architecture
**Master control plane**:

- Master node (1 per node pool)
	- Container runtime
	- etcd
	- kube-scheduler
		- Creates and provisions pods to nodes
	- kube apiserver
		- Frontend for Kubernetes control plane
		- Exposes Kubernetes API
	- kube controller manager
		- Contains multiple controllers for managing Kubernetes service
			- Node controller (monitors nodes)
				- For noticing and responding when nodes, containers or endpoints go down
			- Replication controller (monitors pods)
				- Decides whether new pods have to be created
					- If yes, informs kube scheduler
			- Endpoints controller (monitors services)
				- Endpoint: recipient of any calls between services
				- Maintains a service by managing links between services and the underlying pods on which the service runs
			- Service accound & token controller
	- cloud controller manager
		- For running controllers specific to the cloud provider
		- On-premise Kubernetes clusters (which do not use cloud provider) do not need this
		- Contains controllers such as:
			- Node controller
				- Checks if unresponsive node is deleted in cloud or not
			- Route controller
				- Manages routing of requests
			- Service controller
				- Load balancing within cloud

**Node pool**:

- Worker node (may be multilple)
	- Container runtime
	- kubelet
		- Agent that runs on every node in the cluster
		- Responsible for checking if containers are running within pods in the given node
	- kube proxy
		- A network proxy that runs on each node in the cluster
		- Maintains network rules on all nodes in the cluster
		- Allow network communication to and from the pods (network sessions may be inside or outside the cluster)
