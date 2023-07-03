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
