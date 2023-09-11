# Using Azure storage services

## Storage using Azure disks
### Introduction
Azure Disk Storage (ADS) offers high-performance, highly durable (i.e. failure-resistant) volumes (i.e. block storage) for our workloads (i.e. our applications running on Kubernetes). **_We can mount these volumes as devices on our virtual machines and container instances_**.

Advantageous features of ADS:

- Cost-effective, especially when handling:
	- Unexpected traffic
	- Batch jobs
- Resiliency (i.e. durability) (0% annual failure rate, providing consistent enterprise-level durability)
- Seamless scalability
- Built-in security (automatic and manual encryption available to protect data)

Concepts used by ADS:

- Storage class
- Persistent volumes & persistent volume claims
- Config map
- Environment variables
- Volumes & volume mounts

### Kubernetes-side functions
#### Listing objects to get information
To list available storage classes...

`kubectl get storageclass`

To list available persistent storage claims...

`kubectl get persistentstorageclaim`

#### Procedure
1.
First, create a storage class (_using which pods can request the defined types of storage (defined in the storage class) dynamically_) by creating the required YAML manifest and applying it using the command...

`kubectl apply -f <path of the YAML manifest>`

**NOTE (volume binding options)**: The `WaitForFirstConsumer` option has the advantage (over the default option of `Immediate`) that the allocated storage would be placed in the same physical location as the first pod requesting it, which can reduce memory access latency.

2.
Then, create a persistent storage claim (_using which pods can request storage according to the specified storage class_) by creating the required YAML manifest and applying it using the above command (but for the path in which the PVC's YAML manifest lies).

3.
Then, create a deployment (using a YAML manifest) for the desired application and specify the desired volumes under the "volumes" key (which is an array of one or more volumes). When mentioning a desired volume, we must mention...

- Volume name chosen
- Persistent volume claim to be used to request storage<br>_Alternatively, we can specify a ConfigMap that we want to use to define the specifications of the desired volume. Of course, we should have created the required ConfigMap to begin with.

**NOTE**: We must also make sure to define the volume mounts within the "containers" field of the deployment YAML manifest.

An example of a YAML manifest for such a deployment is given below. This deployment is used to deploy a MySQL database (which is done through a containerized MySQL application), which requires certain storage (i.e. volumes and volume mounts)...

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate 
  template: 
    metadata: 
      labels: 
        app: mysql
    spec: 
      containers:
        - name: mysql
          image: mysql:5.6
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: dbpassword11
          ports:
            - containerPort: 3306
              name: mysql    
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql    
            - name: usermanagement-dbcreation-script
              mountPath: /docker-entrypoint-initdb.d #https://hub.docker.com/_/mysql Refer Initializing a fresh instance                                            
      volumes: 
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: azure-managed-disk-pvc
        - name: usermanagement-dbcreation-script
          configMap:
            name: usermanagement-dbcreation-script
```

> For seeing all the required files necessary for this deployment, check the following link:
> - https://github.com/stacksimplify/azure-aks-kubernetes-masterclass/tree/master/05-Azure-Disks-for-AKS-Storage/05-03-UserMgmt-WebApp-with-MySQLDB/kube-manifests

### Azure-side
Note that deleting a volume provided via Azure Disk Storage through kubectl will not delete the disk storage itself, only the persistent volume object within Kubernetes.

## Azure MySQL database service
The Azure cloud platform offers a fully managed MySQL service that can be connected (using an _ExternalName_ Kubernetes service) to our Kubernetes cluster, thus providing our Kubernetes-based application a backend. An advantage of using Azure's managed MySQL service versus Azure Disk Storage (ADS) is that the latter is bound to a particular container instance (hence pod instance) or virtual machine only (i.e. ADS is allocated for a particular container instance or virtual machine instance only) whereas the Azure MySQL service can be accessed by any pod (hence any container instance) within the cluster by exposing it through the _ExternalName_ Kubernetes service. Furthermore, the Azure MySQL service offers the following features:

- Automatic backup and recovery options
- Automatic upgrading of the software
- High availability (with no extra cost)
- Scalability

### Set-up
Create a MySQL server in the Azure cloud platform. Apply the settings you want, but in particular, to enable the Azure AKS cluster to access the server, toggle the "Allow access to Azure services" option (in the "Connection security" setting of the created MySQL server) to "Yes". Furthermore, if you do not want to enforce an SSL (Secure Socket Link) connection (which is a transport level encryption protocol for data sent over networks like the Internet), toggle the associated option (in the "Connection security" setting of the created MySQL server) to "DISABLED"; enforcing SSL would require additional configurations to allow our applications to connect to the server, which I shall not delve into here.

**Note**: The applied changes would take around 15 minutes to take effect.

### Connecting to Kubernetes cluster
**SIDE NOTE**: _ExternalName service is discussed in "2_kubernetesArchitecture", under the subsection on Kubernetes services._

**STEP 1**:<br>
Create an _ExternalName_ service (by creating and applying a YAML manifest, as discussed in the previous sections).
