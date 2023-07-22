# Introduction to Azure

> To get started with Azure: https://azure.microsoft.com/en-in/.

## Relevance of Azure for Kubernetes
**_First, a recap..._**

Due to the complexity of installing, configuring and managing Kubernetes on a personal computer system, we shall be using Azure as the cloud computing platform to access Kubernetes services. To do this, we shall use AKS (Azure Kubernetes Service), which offers the quickest way to start developing and deploying cloud-native apps (i.e. apps that are stored and run on the cloud) in Azure with built-in code-to-cloud pipelines and guardrails. As a Kubernetes service hosted on the Azure platform, AKS handles critical tasks, like health monitoring and maintenance, and is highly available, secure and fully managed.

**_Why learn Azure..._**

Since we shall be working with Azure, cloud-related concepts -- especially regarding Azure -- will come up at crucial stages in our learning process. Hence, we shall look into Azure and get a general grasp of the platform and key cloud concepts we will need to deal with it effectively.

## Introduction
Azure is a cloud computing platform that offers:

- Infrastructure as a service (IAS)
- Platform as a service (PAS)
- Software as a service (SAS)

## Key tools & concepts
### 1. Azure management levels

a. Tenant
*...manages multiple...*
b. Management groups
*...manages multiple...*
c. Subscriptions (billing occurs here)
*...manages multiple...*
d. Resource groups
*...manages multiple...*
e. Resources (*called Azure services*)
	-  Can be grouped into resource clusters (*collection of resources within a resource group*)

#### e. Resource
An Azure resource is a manageable item (i.e. a computable entity that can be configured, moved, handled and used either automatically or by a user) that is available through the Azure cloud platform. This includes:

- Virtual machines (_in Kubernetes context, this includes nodes_)
- Storage accounts
- Web applications (_in Kubernetes context, this includes pods_)
- Databases
- Virtual networks (_in Kubernetes context, this includes services_)

By definition, a resource can also be:

- Resource group
- Subscription
- Management group
- Tag

> Reference: https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/overview

#### d. Resource group
An  Azure resource group  is a logical group in which Azure resources are deployed and managed. When we create a resource group, we are prompted to specify a location, which is the storage location of

- The resource group metadata
- The space where resources run in Azure (_if you do not specify another region during resource creation_).

> Reference: https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-cli

### 2. Azure CLI & Terraform
Azure CLI (Command Line Interface) is a CLI provided by Azure to manage resource groups and creating, managing and provisioning clusters.

Terraform is the high-level scripting language that is portable, i.e. can be used to access the APIs, services, templates, etc. from any cloud computing platform. It can perform the same functions as Azure CLI, but can be applied for other cloud computing platforms (as well as multiple platforms together) also.

### 4. Azure load balancer

> Reference: https://learn.microsoft.com/en-us/azure/load-balancer/components

A load balancer in general is device or software system that (1) exposes a virtual network (_i.e. set of interconnected nodes, i.e. networked machines -- virtual machines, instances of virtual machines, or physical machines_) to a wider network and (2) distributes the incoming traffic to the virtual network. An Azure load balancer is a load balancer provided by Azure for a cloud-native set of interconnected nodes (i.e. a cloud-native virtual network). Also note that in a cloud-native setting such as Azure, each node is either a virtual machine or an instance of a virtual machine.

#### a. Frontend IP address
This is the IP address of the load balancer itself. It serves as the point of contact between external clients and the virtual network. This IP address can be public or private. If public, then we have an external load balancer (which manages traffic between the virtual network and the wider network). If private, then we have an internal load balancer (which manages traffic within the virtual network).

_Note that a load balancer can have multiple IP's, but we shall not discuss that here_.

#### b. Backend pool
The backend pool is the set of nodes within the virtual network that service the incoming requests that are distributed by the load balancer.

**_In a Kubernetes load balancer context..._**

In the context of a Kubernetes load balancer service in AKS (which automatically creates an Azure load balancer (_i.e. which is a suitable cloud load balancer_) to which the Kubernetes load balancer service is mapped), the backend pool consists of the nodes in our Kubernetes cluster.

#### c. Load balancer rules
A load-balancing rule maps a given frontend IP address and port to multiple backend IP addresses and ports. A load balancer rule is used to define how incoming traffic is distributed to all the instances (i.e. nodes that serve identical roles) within the backend pool. Hence, load balancer rules are for inbound traffic only.

**SIDE NOTE**: _Associating an IP address to a component is called the IP configuration of the component; it means setting the IP address of the component_.

### 5. Azure Kubernetes Service
Azure provides a managed Kubernetes service called Azure Kubernetes Service (AKS) in which Azure manages the master nodes and the end-user's needs in managing the worker nodes.
