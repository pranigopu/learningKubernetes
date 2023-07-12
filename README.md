
# Learning Kubernetes
## References
> - Official documentation for Kubernetes:<br>https://kubernetes.io/docs/home/
> - Udemy course link:<br>https://www.udemy.com/course/azure-kubernetes-service-with-azure-devops-and-terraform 

## Introduction
**Definition 1**: Kubernetes is a (1) portable, (2) extensible and (3) open-source container orchestration system for _automating_ software deployment, scaling, and management.

**Definition 2**: In other words, Kubernetes is an open-source orchestration engine for automating deployment, scaling, and management of _containerized applications_ i.e. applications packaged into containers.

**NOTE**: _Kubernetes is a software platform that can be installed in a computer system (local or remote)._

**SIDE NOTES**:

1.  _The Kubernetes open-source project is hosted by the Cloud Native Computing Foundation._
2. _"Kubernetes" is often abbreviated as "K8s" (i.e. K + 8 letters + s)_
3.  _The name Kubernetes originates from Greek, meaning "helmsman" or "pilot"._

## Containers
Containers are standardized, executable components that combine application source code with the operating system (OS) libraries and dependencies required to run that code in any environment.

In other words, containers virtualize the software layers above the OS level. In other words, a container is an isolated environment for your code, meaning a container has no knowledge of your machine's OS or your files.

Let us compare and contrast containerized app deployment with other app deployment methods...

### Traditional app deployment

- Applications run on physical servers as they would on a physical computer system
- In the case of multiple applications required by an organization, we have the following choices:
	- Multiple applications run on one physical server, often leading to resource allocation, process scheduling and performance issues
	- Multiple applications run on one physical server each, often leading to resource wastage

### Virtualized app deployment

- Virtual machines are created and run on top of physical servers, with possibly multiple VMs running on one physical server's CPU
- Logically, applications are run on mutually isolated VMs
- Benefits
	- Better resource utilization
	- Higher security between applications
	- Scalability, since VMs provide an abstraction from the underlying systems, and hence, present physical resources as a set of disposable VMs

### Containerized app deployment

- Similar to a VM, a container has its own filesystem, share of the physical server's CPU, memory, process space, etc.
- Unlike a VM, a container does not virtualize the entire machine (from the hardware layer to the application layer), but rather, only it only virtualizes the software layers running on top of the OS (i.e. above the OS level)
- Hence, containers can share an OS, and thus, have a more relaxed isolation than VMs
- Furthermore, since containers are not tied to any particular underlying infrastructure (hardware or OS-based), they are portable across systems and OS distributions

#### In summary...
A container is virtualization technology similar to a VM, but instead of virtualizing the entire machine, it only virtualizes the software layers above the OS level. In this way, it packages applications decoupled from any underlying infrastructure, enabling:

- Portability across systems and OS distributions
- Environmental consistency across development, testing, and production, as a containerized application runs the same on a laptop as it does in the cloud
- Resource isolation (similar to VMs), leading to predictable application performance
-   Resource utilization (similar to VMs), leading to high efficiency in terms of memory and CPU usage

Furthermore, it is lightweight as well as easier and more efficient to create and use container images compared to VM images, enabling a more agile development process.

**NOTE**: _Kubernetes does not actually handle the process of running containers on a machine. Instead, it relies on another piece of software called a container runtime._

## Need for container orchestration
In a production environment, you need to manage the containers that run the applications and ensure that there is no downtime. This can involve:

- Service discovery
	- Exposing a container for access
- Load balancing
	- If traffic to a container is high, we must be able to distribute the network traffic across multiple containers while offering the same required functionality
- Storage management within and between containerized applications
- Self-healing
	- This involves restarting containers that fail, replacing containers, killing containers that do not respond to your user-defined health check and not advertising killed containers to clients until they are ready to serve
- Managing configuration of applications (including network applications running on multiple containers)
- Managing sensitive information
	- Storing and managing "secrets", i.e. sensitive information such as passwords, authorization tokens, and SSH keys

As a container orchestration platform, Kubernetes offers the above services and more.

## Declarative vs. imperative programming in Kubernetes
The Kubernetes engine supports both declarative and imperative methods for programming instructions.

Imperative methods...

- Command line interface (CLI)

Declarative methods...

- YAML (Yet Another Markup Language) manifests
- Terraform scripting language

## The approach I am taking...
Due to the complexity of installing, configuring and managing Kubernetes on a personal computer system, we shall be using Azure as the cloud computing platform to access Kubernetes services. To do this, we shall use AKS (Azure Kubernetes Service), which offers the quickest way to start developing and deploying cloud-native apps (i.e. apps that are stored and run on the cloud) in Azure with built-in code-to-cloud pipelines and guardrails. As a Kubernetes service hosted on the Azure platform, AKS handles critical tasks, like health monitoring and maintenance, and is highly available, secure and fully managed.
