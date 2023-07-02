# Docker
## Relevance of Docker in Kubernetes
Kubernetes is a container orchestration engine, but it does not build and deploy containers directly. Even at its most basic level, it deals with "pods", which exist in a layer of abstraction above containers (_in other words, in Kubernetes, containers are wrapped into pods, and pods are the basic unit of software for Kubernetes_). Hence, there is a need for software to actually build and deploy the containers themselves. This is where Docker comes in. In practice, Kubernetes is the engine that automates and expands on the operations of Docker.

## Introduction
**Docker** is a set of open source platform-as-a-service products that use OS-level virtualisation (also called containerisation) to deliver software in packages called containers. **Docker Engine** is the software platform (that runs on top of an OS ) that hosts the containers, allowing you to build, test, and deploy applications via containers. Containers have the necessary software environment for the applications to run, i.e. they have everything the applications need to run including libraries, system tools, code, and runtime.

### Notes on above terms
OS-level virtualisation (also called containerisation) is the creation by the kernel of one or more isolated instances (i.e. software-level recreations) of the user-space. It abstracts the physical resources of the system by provisioning them into one or more instances of the user-space.

A user-space is the set of software elements (libraries, system tools, code,, and runtime etc.) used by an OS to interact with the kernel, so creating multiple instances of the OS also creates multiple instances of the user-space. Hence, note:

_In OS-level virtualisation does not recreate the OS itself, only the user-space, i.e. the software layers that run above the OS._


## Need for containers
This is a kind of recap and expansion on the section on containers in the README file.

### Traditional app deployment infrastructure

- Level 1: Hardware Infrastructure
- Level 2: Operating System
- Level 3: Libraries + Dependencies
- Level 4: Services (WebServers, AppServers, Databases)

Problems:

- Installation & configuration for every server
- Compatibility & dependencies
- Inconsistencies across environments
- Operational support (_i.e. more resources needed to handle day-to-day operational issues_)
- Provisioning resources securely (ex. in role-based access control) to specific environments (ex. developer environment) is challenging

### Virtualised app deployment infrastructure

- Level 1: Hardware infrastructure
- Level 2: Hypervisor
- Level 3: Virtual Machine (herein, levels 2-4 of the traditional deployment infrastructure are implemented)
	- Level 3.1: Operating System
	- Level 3.2: Libraries + Dependencies
	- Level 3.3: Services (WebServers, AppServers, Databases)

Problems:

- Compatibility & dependencies
- Inconsistencies across environments

### Dockerised app deployment infrastructure
(_A type of containerised app deployment_)

This can be applied within physical app deployment infrastructure or virtualised app development infrastructure. In either case, the Docker runs on top of the OS level. There can by multiple Docker containers in a single server or virtual machine.

- Level X: Operating System
- Level X+1: Docker
- Level X+2: Containers
	- Level (X+2).1: Libraries + Dependencies
	- Level (X+2).2. Service (usually a single service or smaller set of services)

Why containers?

- Flexible
- Portable
- Lightweight
- Scalable
- Loosely coupled to infrastructure, i.e. self-sufficient
- Secure

## Docker architecture
### Related to management of Docker objects

- Docker daemon (`dockerd`)
	- Listens for Docker API requests
	- Manages Docker objects, such as
		- Images (discussed later)
		- Containers
		- Networks
		- Volumes (persistent storage)
- Docker client (`docker`)
	- An optional part of the architecture
	- Abstracts access to Docker daemons
		- Primary way for many users to interact with Docker
	- Commands run through it are sent by it to Docker daemons
		- Docker client can communicate with multiple daemons
		- Client uses Docker API to interact with daemons
- Docker host
	- The machine on which Docker is being run
	- Can be virtual or physical

### Related to Docker containers

- Docker image
	- A read-only template containing instructions to create container
	- Images are often based on other images with customisations
- Docker container
	- Check the definition of "container" in software
	- In short, it is an isolated environment (complete with libraries and dependencies) used to package codes or applications to be run
	- Through Docker API or Docker CLI (command line interface), we can create, start, stop, move or delete containers
	- We can do the following to containers
		- Connect them to one or more networks
		- Assign storage to them
		- Create new image using their current state
	- A deleted container's state -- if not stored in persistent storage -- is also lost from memory
- Docker registry
	- A storage to store Docker images
	- Can be private or public
	- Through the client, we can use the following commands:
		- pull: access Docker image from configured registry
		- push: store Docker image to configured registry
	- **Docker Hub**
		- This is a public Docker registry and can be accessed and used by anyone
		- By default, Docker looks for images in Docker Hub

## Docker installation
### Supported platforms
The Docker Engine is supported on a variety of software platforms...

- Desktop platforms (for desktop-level usage)
	- Mac
	- Windows
- Server platforms (for server-level usage)
	- CentOS
	- Debian
	- Fedora
	- Ubuntu

Check: [Installation documentation](https://docs.docker.com/engine/install/)
