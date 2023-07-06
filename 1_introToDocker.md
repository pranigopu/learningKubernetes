# Docker
## Relevance of Docker for Kubernetes
Kubernetes is a container orchestration engine, but it does not build and deploy containers directly. Even at its most basic level, it deals with "pods", which exist in a layer of abstraction above containers (_in other words, in Kubernetes, the basic units of software are containers are wrapped into objects called pods_). Hence, there is a need for software to actually build and deploy the containers themselves, and this is the kind of software Docker provides. Kubernetes is the engine that expands on the results of a container engine, with the most popular container engine being Docker.

## Introduction
**Docker** (_speaking of the platform as a whole_) is a set of open source platform-as-a-service products (_ex. Docker Desktop, Docker Scout, Docker Hub, etc._) that use OS-level virtualization (also called containerization) to deliver software in packages called containers. **Docker Engine** is the software platform (that runs on top of an OS ) that hosts the containers, allowing you to build, test, and deploy applications via containers. Containers have the necessary software environment for the applications to run, i.e. they have everything the applications need to run including libraries, system tools, code, and runtime.

### Notes on above terms
OS-level virtualization (also called containerization) is the creation by the kernel of one or more isolated instances (i.e. software-level recreations) of the user-space. It abstracts the physical resources of the system by provisioning them into one or more instances of the user-space.

A user-space is the set of software elements (libraries, system tools, code,, and runtime etc.) used by an OS to interact with the kernel, so creating multiple instances of the OS also creates multiple instances of the user-space. Hence, note:

_In OS-level virtualization does not recreate the OS itself, only the user-space, i.e. the software layers that run above the OS._


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

### Virtualized app deployment infrastructure

- Level 1: Hardware infrastructure
- Level 2: Hypervisor
- Level 3: Virtual Machine (herein, levels 2-4 of the traditional deployment infrastructure are implemented)
	- Level 3.1: Operating System
	- Level 3.2: Libraries + Dependencies
	- Level 3.3: Services (WebServers, AppServers, Databases)

Problems:

- Compatibility & dependencies
- Inconsistencies across environments

### Dockerized app deployment infrastructure
(_A type of containerized app deployment_)

This can be applied within physical app deployment infrastructure or virtualized app development infrastructure. In either case, the Docker runs on top of the OS level. There can by multiple Docker containers in a single server or virtual machine.

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
	- Images are often based on other images with customizations
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
		- It is a repository hosted by the Docker Platform for finding and sharing container images
		- By default, Docker looks for images first locally and then in Docker Hub

### Docker platform components
- Docker Desktop
	- The downloadable object-code of the desktop client software component of the Docker Platform
- Docker Scout
	- An added component of the Docker Platform used for providing insights and suggestions on improving software supply chain, ex. security, vulnerability scanning.
- Telepresence for Docker
	- An added component of the Docker Platform used to connect Docker Desktop to a remote Kubernetes cluster, using certain _Ambassador Labs_ third-party products.
- Docker Platform or Service
	- The Docker subscription service software + components ordered by customer, including Docker Desktop, Docker Hub, Docker Scout, Telepresence for Docker, and any add-on services, as well as any updates

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

> Check: [Installation documentation](https://docs.docker.com/engine/install/)

## Basic commands
The keyword "docker" refers to the client.

### Checking Docker Engine version...
View Docker Engine's version...

`docker version`

### Login to + logout of Docker Hub...
Login...

`docker login -u <username> -p <password>`

Logout...

`docker logout`

### Pulling Docker image from Docker Hub...
`docker pull <image name>`

This downloads the Docker image into your system, thus being accessible by the Docker Engine running on your system, i.e. the image will be a part of the host's software environment.

### Creating + starting Docker container...
Running the downloaded image. This will create and start the container based on the image, and thus, this will run the application within the container...

`docker run --name <container name> -p <local port to run container app> -d <image name>`

The `--name` argument is to give a chosen name to the created and started container, and it is optional; if you do not specify a name for the container the system will pick one instead. The container's application will run on the specified port number of the system and the IP address of this running instance of the application would be created accordingly (system address ("localhost", if run locally) + port number).

### Stopping and starting a created container...
`docker stop <container name or ID>`

`docker start <container name or ID>`

The `docker run` command creates and starts a new container, while the `docker stop` and `docker start` commands deal with already created containers. To restart (i.e. stop then automatically start) container:

`docker restart <container name or ID>`

### Listing running containers...
`docker ps`

The options for this command are:

- `docker -a`
	Lists all created containers, not only running containers. However, it does not show removed containers, only existing containers.
- `docker -q`
	This lists only the container IDs.

### Listing downloaded images...
`docker images`

This will also list each image's ID.

### Checking other container-related information...
View live statistics of container resource usage...

`docker stats`

Display running processes of a container...

`docker top <container name or ID>`

View port mapping of a particular container...

`docker port <container name or ID>`

### Removing (i.e. deleting from host) a container...
`docker rm <container name or ID>`

Note that you must first stop the container before attempting to remove it;  the daemons will prevent your from removing running containers. However, you can force the removal of a running container using the `-f` option:

`docker rm -f <container name or ID>`

### Removing (i.e. deleting from host) a downloaded image...
`docker rmi <image ID>`

### Opening the container's terminal...
_**(i.e. a terminal running within the container's environment)**_

`docker exec -it <container name or ID> ...`

Look up the exact command structure for your particular platform. Also, once in the container's terminal, you can exit it using the `exit` command.
