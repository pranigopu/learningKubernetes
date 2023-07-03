# Kubernetes architecture
## Basic terms
### Pod
A pod is a logical wrapper within  Kubernetes for one or more containers. Just as a container is the basic software unit of Docker, so is pod the basic software unit of Kubernetes. Containers are encapsulated within pods. Some points on pods...

- A pod is the smallest object creatable in Kubernetes
- Pods usually have 1-1 relationship with containers
	- Pods may have multiple containers (rare cases)
	- There cannot be multiple identical containers in one pod
- Scaling
	- Scale up: creating identical pods
	- Scale down: deleting identical pods
	- No scaling within pods (i.e. no adding containers within pods), only scaling with pods (i.e. adding pods to the cluster)

### Node & cluster
A node is a machine (physical or virtual) which provides the platform for pods to run on. A cluster is a collection of nodes managed by a control plane (discussed further


### ReplicaSet
A ReplicaSet (RS) is a Kubernetes object that acquires and maintains (by creating or deleting) a specified number of replicas of specific kind of pod within a node or cluster of nodes.

### Deployment

### Service
An abstraction for pod, sitting on top of pods and acting as a load balancer while keeping a stable virtual ID.

### Imperative vs. declarative configuring/deploying applications in Kubernetes

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
