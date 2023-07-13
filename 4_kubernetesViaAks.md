# Kubernetes via AKS
**NOTE + RECAP**: Azure provides a managed Kubernetes service called Azure Kubernetes Service (AKS) in which Azure manages the master nodes and the end-user's needs in managing the worker nodes.

## Tools used
### Kubernetes-specific tools
#### kubectl
The Kubernetes command-line tool **kubectl** enables you to run commands for Kubernetes clusters. You can use kubectl to...

- Deploy applications
- Inspect and manage cluster resources
- View activity logs

**NOTE**: When working with a cloud-native Kubernetes cluster using our local system, kubectl is used to pass commands to the cloud provider's API, which in turn passes commands to the cloud-native Kubernetes API. To do this, we need certain modules (discussed later) that enable this communication via the local system's command line interface.

## Accessing & managing resources in an AKS cluster

### Configure AKS cluster credentials in local system
_Note that an AKS cluster is simply a cloud-native (specifically a Azure-native) Kubernetes cluster_.

`az aks get-credentials --resource group <resource group name> --name <cluster name>`<br>OR<br>`az aks get-credentials -g <resource group name> -n <cluster name>`

Here, we want to configure our context (i.e. the work environment context) so that implicit references to a Kubernetes cluster resolve with the above given AKS cluster. We give resource name merely to locate and identify the cluster.

I have created the cluster `aks-demo-pg` within the resource group `aks-pg-rg`. I shall be using these names for the configuration.

**_Results..._**

az module gives the following warning:

`az : WARNING: Merged "aks-demo-pg" as current context in C:\Users\prana\.kube\config`

This means the `config` file here is currently set to refer to the cluster `aks-demo-pg`.

### Nodes & pods

#### View nodes in the cluster

`kubectl get node`<br>OR<br>`kubectl get node`

**_Results..._**

```
NAME                                STATUS   ROLES   AGE    VERSION
aks-agentpool-19544217-vmss000003   Ready    agent   136m   v1.25.6
```

`kubectl get node -o wide` gives additional information about the nodes.

#### Create & view a pod

`kubectl run <pod name chosen> --image <container image name>`

In my case, I will use the image `stacksimplify/kubenginx:1.0.0` from Docker Hub. I will name my pod "pod1". Pod name can have hyphens as well.

`kubectl run pod1 --image stacksimplify/kubenginx:1.0.0`

**_Response message..._**

`pod/pod1 created`

##### View created pods

To view all created pods...

`kubectl get pod`<br>OR<br>`kubectl get pods`

**_Results..._**

```
NAME   READY   STATUS             RESTARTS      AGE
pod1   0/1     CrashLoopBackOff   5 (25s ago)   3m29s
```

As with `kubectl get nodes`, we can add the `-o wide` option, i.e. we can give `kubectl get pods -o wide` to get more information about the created pod(s), such as its internal address and host node.

We can also get particular pod(s) using the following...

`kubectl get pod <pod name 1> <pod name 2> ...`<br>OR<br>`kubectl get pods <pod name 1> <pod name 2>`

#### View created pods in detail

Using the command...

`kubectl describe pod <pod name 1> <pod name 2> ...`<br>OR<br>`kubectl describe pods <pod name 1> <pod name 2>`

... we can get a detailed look into the given pod. The extra information we get here includes:

- Namespace
- Priority
- Service account responsible for it
- Containers present + their information
- State (ex. waiting, ready, running, etc.)
- Events related to the pod

To get a description of all pods in the cluster at once, do not give any arguments (i.e. any pod names.

#### View pod logs

Kubernetes captures logs from each container in a running pod. Using the command...

`kubectl logs <pod name>`

... we can view the Kubernetes API requests (_which happen to be HTTP requests and responses_) and responses to and from the container(s) running on the pods. We can view the live streamed logs of the pod's container(s) using the `-f` option as follows...

`kubectl logs -f <pod name>`

**SIDE NOTE** (example of an API request that would be logged):<br>_To externally access an application running on a pod (using an external IP address), we need to expose the pod using a load balancer service (discussed later). Once we do so, we can view the details of this service to obtain the external IP address of the service through which to access the application. Using this IP address, we can send an HTTP "GET" request to access the application, for example on a web browser. Such a request would be logged for the pod's container (in which the application is running)_.

#### Connect to a container running on the pod

To connect to a container running on a pod, we enter a command-line interface within the container's environment (enabling us to give commands to the container's application). To do this, we use the following command...

`kubectl exec -it <pod name> <container name> -- bin/bash`

Here, we are executing the Bash terminal (the CLI of the container) of the container, enabling us to give commands to the container. If there is only one container in the pod (as is usually the case), simply do...

`kubectl exec -it <pod name> -- bin/bash`

Alternatively, if you want to enter individual commands to the container without entering its CLI, you can do (assuming you have only one container)...

`kubectl exec -it <pod name> -- <bash command>`

#### Overview of ways to interact with pods

- Describe command
- Logs command
- Connecting to container

### Overview of some commands
#### Describe & get commands
We saw above how describe and get commands can be used for pods. But in fact, these commands apply for any resource type in the cluster (nodes, pods, services, etc.). The general structure of the commands is as follows...

To get all instances of a resource type...

`kubectl get <resource type>`

To get particular instances of the resource type...

`kubectl get  <resource type> <instance name 1> <instance name 2> ...`

The same format applies for `kubectl describe` also. Yes, we can either describe particular instances or all instances at once. Note that it does not matter whether `resource type` is given in singular or plural form; the command works the same either way. However, using different spellings may improve readability or conceptual clarity.

To get all instances of all resource types (works for describe also)...

`kubectl get all`

#### More on get command
The "get" command has the following options (given after an `-o` being appended after the required arguments):

- `kubectl ... -o wide`<br>Gives extra information on the object(s).
- `kubectl ... -o yaml`<br>Gives the YAML manifest(s) of the object(s).

#### Delete command
The general structure...

`kubectl delete <resource type> <instance name 1> <instance name 2> ...`

As always, whether the resource type is given in singular or plural does not matter. An example for the use of `delete` command...

`kubectl delete pod pod1`

Response message...

`pod "pod1" deleted`

### Services
We can expose (i.e. make available and accessible) an application running on a pod or a set of pods by using various services. We can expose the application either internally (i.e. to nodes within the cluster) or externally. To expose applications internally, we use the _cluster IP service_ (provides communication between pods in within the cluster). To expose applications externally (i.e. allow access from outside the cluster), we need to create either a _node port_ (currently in preview), an _ingress service_ or a _load balancer service_ (a service created by AKS for the cluster by default). Since we are working with AKS, we shall discuss Kubernetes services with respect to an AKS environment.

#### Load balancer service

> Additional references:
> - https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/
> - https://kubernetes.io/docs/concepts/services-networking/service/

When creating a service, we have the option of automatically creating a cloud load balancer (i.e. a load balancer that exposes the cluster to the Internet (_or any IP-based network_) via its own unique public IP). This provides an externally-accessible IP address that sends traffic to the correct port on the cluster's nodes (i.e. the correct port on the cluster nodes through which the desired applications are running), _provided your cluster runs in a supported environment (such as AKS) and is configured with the correct cloud load balancer provider package_.

**_AKS specific implementation..._**

When working with AKS specifically, creating a Kubernetes load balancer service for our cluster automatically creates an Azure standard load balancer (SLB). Now, note that in Kubernetes, pods are exposed to a network via the Service API, which is a part of the Kubernetes API itself. When a service is created, a "Service" object is created that (1) maintains the configurations of the desired service and (2) maintains the service by handling requests and responses to and from the Service API. In this way, the Kubernetes load balancer service is eventually mapped to the cloud load balancer created for the cluster -- an Azure SLB instance in this case -- via the Service API. The cloud load balancer is what ultimately exposes the cluster through its IP address. This approach to exposing the pods in a cluster ensures:

- Decoupling the Kubernetes service from the specific cloud infrastructure, enabling portability across cloud providers
- A security layer between the cloud platform and the Kubernetes cluster

**_Upon deployment..._**

Upon deploying the Kubernetes load balancer service, the Azure SLB will do the following:

- Creates a public IP address that is configured as the frontend IP address
- Creates a new load balancer rule to associate frontend IP address with the backend pool (_i.e. the nodes in the cluster in our case_)

Note that creating a cluster in AKS automatically creates a Kubernetes load balancer service, hence automatically initiating the above processes.

##### Exposing a set of pods through a load balancer service
In creating a service, what we are really doing is _exposing_ a specific set of pods while setting a specific policy with which to access them. This also applies to a load balancer service. To "create" a service...

`kubectl expose pod <pod name 1> <pod name 2>... --type=<type of service> --port=<port number on which service is to run> --name=<service name>`

In particular, we shall do...

`kubectl expose pod pod1 --type=LoadBalancer --port=80 --name=service1`

(_The set of pods being exposed here is just a single pod, "pod1"_)

**_Response message..._**

`service/service1 exposed`

**_Viewing service details (using get command)..._**

```
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
kubernetes   ClusterIP      10.0.0.1       <none>          443/TCP        5d23h
service1     LoadBalancer   10.0.194.238   20.235.63.146   80:30253/TCP   84s
```

**NOTE 1**: The ClusterIP service is a default service created for internal load balancing. Hence, unlike our LoadBalancer service, it does not have an external IP address.

We can access "service1" from outside the cluster using the external IP address. What we are accessing through this service is in fact the application(s) running on the pod(s) exposed by this service. In this case, we are accessing the application in "pod1" as exposed by "service1".

**NOTE 2**: Unlike pods, services are persistent, i.e. their data is preserved even if the cluster is restarted. Hence, though the pods they expose are destroyed upon restarting the cluster, the services retain the names of the pods they were meant to expose.

#### More on the expose command
A service can expose any resource within the cluster, not just pods but also ReplicaSets, deployments, etc. The general structure of the command is...

`kubectl expose <resource type> <instance name or names> --type=<type of service> --port=<port number for exposure> --name=<service name chosen>`

**NOTE**: When exposing objects that abstract the pods involved (such as ReplicaSets), the service used is configured to expose an application (with the specific pods being abstracted as endpoints).

### ReplicaSets
(**_Includes a recap from the discussion on Kubernetes architecture's basic concepts_**)

A ReplicaSet (RS) is a resource object that acquires and maintains (by creating or deleting) a specified number of replicas of specific container(s) within a node or cluster. It is used to guarantee the availability of a specified number of identical containers (hence, identical instances of the application(s) running on the containers).

The purpose of a ReplicaSet is to maintain a stable set of running replicas of a container (i.e. running instances of an application) at any given time. _Note that each replica of a container is also a container itself, and like all containers in Kubernetes, each replica is wrapped in a replicated pod_. In practice, we provide a container image for the ReplicaSet to replicate; the container is created and replicated by the ReplicaSet according to the way we define the ReplicaSet.

ReplicaSets do the following:

- Ensure high availability and reliability of applications (_by creating and maintaining replicas of certain container(s), which serve as backups in case of failure_)
- Enable seamless scaling (_by creating or deleting replicas as needed_)
- Load balancing (_by distributing requests appropriately to the replicas_ **_using services that are a part of the ReplicaSet_**)

#### Creating a ReplicaSet
There is no imperative method (hence no commands) to create a ReplicaSet. Instead, we must use YAML (Yet Another Markup Language) manifest (a declarative method) to create it.

The YAML manifest must contain various key details, such as:

- Metadata (ex. name to be give to ReplicaSet,)
- Number of replicas to be maintained
- Containers to replicate

I have used the following YAML manifest (taken from https://github.com/stacksimplify/azure-aks-kubernetes-masterclass/tree/master/03-Kubernetes-Fundamentals-with-kubectl/03-02-ReplicaSets-with-kubectl):

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset1
  labels:
    app: my-helloworld
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-helloworld
  template:
    metadata:
      labels:
        app: my-helloworld
    spec:
      containers:
      - name: my-helloworld-app
        image: stacksimplify/kube-helloworld:1.0.0
```

_As in every previous example, I shall be using the container image_ `stacksimplify/kubenginx:1.0.0`, _which present in Docker Hub_.

With the above script, we initiate the creation of replica containers from the given image (in the last line), which of course means (as we are in a Kubernetes context) the creation of replica pods. From the above, note that we connect the ReplicaSet to the containers it must replicate not by reference to specific pods but by reference to the applications themselves, or more specifically, the applications running in specific containers. In other words, the way a ReplicaSet is connected with pods is abstracted from us the end users.

To create a ReplicaSet using the above YAML manifest, we do the following...

`kubectl create -f <file path>`

The "file path" is the path of the YAML file we have stored in our local system (since we are working from our local system), To be more explicit, we have to create and store the required YAML file in our local system and access it from within our filesystem using its path.

**SIDE NOTE** _When using kubectl to interact with a cloud-native cluster, since we are using kubectl to access and interact with the cloud provider's API, we can run kubectl no matter what our present working directory in our local system is. Hence, to access the YAML file more easily, we can navigate in the CLI to the directory it is in and simply give the file name as a path_.

**_Like with any other Kubernetes object, we can deal with the created ReplicaSet using the kubectl "get", describe" and "delete" commands._**

#### The pods created by the ReplicaSet
The replica pods created by the ReplicaSet "belong" to the ReplicaSet. This fact is apparent in their auto-generated names, which have the form:<br>`<replica set name>-<identifier>`. These pods can be accessed just like any other pods in the cluster.

##### Pods replicated automatically to maintain high availability & reliability
If certain pods of the ReplicaSet are destroyed or deleted, it will automatically create the right amount of replica containers (hence replica pods) to fulfill the required number of replicas.

#### Scaling a ReplicaSet
We use the "scale" command as in the following...

`kubectl scale --replicas=<enter number> replicaset <ReplicaSet name>`

This changes required number of replicas to be maintained by the ReplicaSet.

#### Updating (or replacing) a ReplicaSet
##### Imperative method
To update the image of the ReplicaSet's container (_i.e. the source container which is to be replicated_), i.e. keep the container name while changing its image, we use the `set image` command as in the following...

`kubectl set image replicaset <ReplicaSet name> <container name>=<new container image>`

##### Declarative method
You can change the properties of a ReplicaSet by recreating, replacing and redeploying it by editing the YAML file or choosing a new YAML file and applying the following command...

`kubectl replace -f <file path>`

**NOTE**: "replace" only works if there exists a ReplicaSet in the cluster that has the same name as the one given in the updated or new YAML manifest. The YAML file itself may be completely changed (apart from the chosen ReplicaSet name), renamed or relocated, but as long as the right name is given, the replacement will be done.

For an example of an update, we can scale up or scale down the ReplicaSet by increasing or decreasing the replica requirement (i.e. the `replicas` field in the YAML manifest before applying the above mentioned command.

###### TIP: Opening the ReplicaSet's YAML file via CLI to edit
To edit the ReplicaSet's YAML file (in order to update it), we enter the following...

`kubectl edit replicaset <ReplicaSet name>`

Upon entering this command, we will get the ReplicaSet's YAML file opened in our computer's preferred text editing software (such as Notepad, VI, etc.). The command's execution terminates upon closing the file. **_Doing this directly updates (i.e. replaces) your ReplicaSet, removing the need for the "replace" command_**.

#### Exposing a ReplicaSet using a service
While a ReplicaSet can act as a load balancer for its pods for requests within the cluster, it cannot be used to expose its pods externally. To do the latter, we need to use the right services  (such as a LoadBalancer service). In other words, you need to expose a ReplicaSet through a service in order to access the ReplicaSet (or rather the applications running on its pods) from outside the cluster (ex. through your web browser, using the service's external frontend IP address). The "expose" command required to do this task has been previously discussed in the section on services.

##### Advantage of exposing ReplicaSets instead of particular pods
Pods are not persistent, i.e. the are destroyed upon restarting the cluster. However, when exposing a ReplicaSet, the service's definition makes no references to specific pods but the application itself (the application being the one from the container image through which containers (hence pods) were created and replicated). A ReplicaSet is persistent, hence the references in the service remain intact despite cluster restarts.

#### Final notes on ReplicaSets
Like services, ReplicaSets are persistent, i.e. the are maintained even if the cluster has been restarted. Its previous pods would be destroyed, but it would automatically create new pods upon restart.

### Deployments
(**_Includes recap of the discussion on Kubernetes architecture's basic concepts_**)

A deployment is a resource object that provides declarative updates to applications (i.e. to containerized applications running on pods or ReplicaSets). To elaborate, we can describe a desired state (i.e. a desired combination of properties for the application(s) being deployed) and the **deployment controller** changes the actual state of the targeted application(s) and associated environment to the desired state _at a controlled rate_.

**NOTE 1**: Any changes being applied to the deployment will be immediately applies to the application. Such a change generally involves the creation of new pods or ReplicaSets and the termination of old pods or ReplicaSets, in order to align with the deployment's new specifications.

**NOTE 2**: As a deployment is updated, it maintains a number of its older versions (usually 10 older versions) to which we can roll back when we want.

We can do the following with deployments:

- Roll out a ReplicaSet through a deployment (_happens by default when creating a deployment_)
- Update a deployment (_i.e. update applications and their environment_)
- Roll back to older versions of a deployment
- Roll forward to newer versions of a deployment
- Scale a deployment (_similar to scaling a ReplicaSet_)
- Pausing and resuming a deployment (_to make changes without simultaneously creating and destroying pods_)
- Observe the deployment status of the application(s) being deployed
- Canary deployments (_i.e. distributing traffic to new and old versions of the deployment, in order to test the newer version_)

#### Creating a deployment
Unlike with ReplicaSets, deployments can be created through imperative methods (i.e. commands). We use the following...

`kubectl create deployment <deployment name chosen> --image=<container image>`

As with ReplicaSets, we provide a container image for which to create a deployment. The command initiates the creation of the container and thereby the containerized application (_which we want to deploy_), and associates it to the deployment object being created. _As in every previous example, I shall be using the container image_ `stacksimplify/kubenginx:1.0.0`, _which present in Docker Hub_.

#### Scaling a deployment
Similar to what we used for ReplicaSets, we use the following...

`kubectl scale --replicas=<enter number> deployment <deployment name>`

This changes required number of replicas (_of the containerized application, thus the containers, thus the pods_) to be maintained by the ReplicaSet. Note that updating the deployment simply updates (i.e. replaces) the ReplicaSet associated to it, maintaining the same name for the ReplicaSet.

**NOTE**: In practice, we usually use autoscaling methods to scale the deployment. We shall discuss this later.

#### Updating a deployment
##### Imperative method
Similar to what we did for ReplicaSets, to update the image of the deployment's ReplicaSet's container (_i.e. the source container which is to be replicated_), i.e. keep the container name while changing its image, we use the "set image" command as in the following...

`kubectl set image deployment <deployment name> <container name>=<new container image>`

##### Declarative method
To do this, we shall be opening the deployment's YAML file via CLI to edit it, similarly to how we did it for ReplicaSets. To do so (in order to update it), we enter the following...

`kubectl edit deployment <deployment name>`

Upon entering this command, we will get the deployment's YAML file opened in our computer's preferred text editing software (such as Notepad, VI, etc.). The command's execution terminates upon closing the file.

##### Viewing the rollout history
Every update to the deployment (including edits) is counted as a rollout, and the creation of the deployment is also counted as a rollout. We can see the rollout history (i.e. update history) of our deployment using...

`kubectl rollout history deployment <deployment name>`

##### Verifying the rollout status of the deployment
To see state of the most recent update (i.e. rollout) of the deployment, we use...

`kubectl rollout status deployment <deployment name>`

#### Exposing a deployment using a service
Just like ReplicaSets, deployments can be used to expose application(s) and the associated pods internally (i.e. within the cluster). But to expose them externally (i.e. to make it accessible outside the cluster), we need to use the right services  (such as a LoadBalancer service). The "expose" command required to do this task has been previously discussed in the section on services.

##### Advantage of exposing deployments instead of particular pods
Exposing deployments has the same essential advantage as exposing ReplicaSets (_discussed in the previous section_).

## Some extra notes
### Separating resource type names and instance names in kubectl
When mentioning resource types and their instances (in any command; get, describe, scale, set...), we can either use a space or a slash to separate the resource type name and the instance name(s). For example...

`kubectl get pod/pod1`

... is the same as ...

`kubectl get pod pod1`
