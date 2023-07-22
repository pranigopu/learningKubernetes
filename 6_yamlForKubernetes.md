
# YAML for Kubernetes
**(YAML, i.e. "Yet Another Markup Language" or "YAML Ain't a Markup Language")**

## Introduction
To be clear, **_YAML is not a markup language_**. A **markup language** is a coded means (i.e. a method that uses specific symbols and syntax) to control a document's structure, formatting and the relationships between its parts. YAML, however, is a data serialization language. **Data serialization** is the process of translating a data structure or object into a format that can be stored (ex. files in secondary storage devices, data buffers in primary storage devices) or transmitted (ex. data streams over computer networks) and reconstructed later (possibly in a different computer environment).

> References: https://en.wikipedia.org/wiki/Serialization

### Purpose of YAML in Kubernetes
YAML is a human-readable data serialization language, i.e. a language used to define data serialization with respect to certain data structures or objects (ex. Kubernetes objects such as pods, services, etc.). In Kubernetes, YAML is used to program a Kubernetes cluster declaratively (i.e. by describing the desired properties) rather than imperatively (i.e. by giving commands to initiate the desired processes).

### Features of YAML
- Similar to JSON (ex. uses key-value pairs to store data)
- Designed to be clean and easy to read
- Valid extensions for YAML files are:
	- ".yml"
	- ".yaml"

### Basic topics in YAML to be learnt

- Key-value pairs and comments
- Dictionaries (or maps)
- Arrays (or lists)
- Spaces
- Document separator

## Basics

### Key-value pairs
Key-value pairs denote the smallest pieces of data that can be represented in a YAML file. Key-value pairs are represented in the following format...

`<key>: <value>`

_The space after the colon is mandatory_.

To comment, use the hash ('#') at the start of the line or part of the line to be commented (like in Python), as in...

`# Commented...`

### Dictionaries (or maps)
A dictionary or a map is a set of properties (denoted by key-value pairs) grouped under a key (called an item in this context). This maps these properties to the item. The format is...

```
<item name>:
	<key 1>: <value 1>
	<key 2>: <value 2>
	...
```

For example...

```
person:
	name: Bing-Bong
	age: 23
	profession: wrestler
```

The grouping of the properties under the item name is denoted by the indentation (tab-indentation, which must be precise, as in Python).

### Arrays (or lists)
An array or a list is a set of values grouped under a single key. This associates the key (the array name in this context) to several values (not properties, as in a dictionary). The format is...

```
<key>:
	- <value 1>
	- <value 2>
	...
```

For example...

```
fears:
	- spiders
	- heights
	- enclosures
```

As always, the right indentation is needed (tab-indentation) before the hyphens. Also, a space must follow the hyphens (i.e. before the value is given). Note that each value in the array can itself be a value, a key-value pair, a dictionary or an array.

### Document separator
A single YAML file can be used to contain the data serialization of multiple data structures or objects, i.e. a single YAML file can contain multiple declarative documents. However, to do this, we need a document separator between the different "documents" within the YAML file. The document separator is

`---`

i.e. three consecutive hyphens in its own line. In Kubernetes context, we can use this feature to create multiple Kubernetes objects by applying a single YAML file.

## Creating Kubernetes objects using YAML
### Basic definitions
For any Kubernetes object, we include the following information at the start of the document...

- **apiVersion**: The version of Kubernetes API used by object
- **kind**: The type of Kubernetes object being defined, ex. pod, service, etc.
- **metadata**: The metadata for the Kubernetes object, containing:
	- Name (of the Kubernetes object) (_optional_)
	- Labels (for further identification of object) (_mandatory for pods, optional for others_)
	- Namespace (_generally given as "default", in which case it need not be mentioned_)
- **spec**: The actual definition of the object

**SIDE NOTE**: _Names and labels are lowercase alphanumeric sequences starting with an alphabet, and may include hyphens_.

### Examples for YAML definitions of Kubernetes objects
#### Pods

```
apiVersion: v1
kind: Pod
metadata:
	name: pod1
	labels:
		app: app1 # Generally containerized into a single container
spec:
	containers: # List
		- name: app1
		  image: stacksimplify/kubenginx:1.0.0
		  ports:
			  - containerPort: 80
```

#### LoadBalancer service

```
apiVersion: v1
kind: Service
metadata:
	name: service1
spec:
	type: LoadBalancer # Type of service
	selector: # Defines service's backend (i.e. what the service must expose)
		app: app1
	ports: # For creating mapping between service frontend and backend
		name: http
		port: 80 # Service port, i.e. frontend port
		targetPort: 80 # Containter port, i.e. backend port
```

#### ReplicaSet & deployment
**ReplicaSet example...**

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset1
  labels:
    app: my-helloworld
spec:
  replicas: 3 # The number of replicas of the app
  selector:
    matchLabels: # The labels (ex. app names) to be subsumed under the ReplicaSet
      app: my-helloworld # The ReplicaSet will be used to run this app
  template:
    metadata:
      labels:
        app: my-helloworld
    spec:
      containers:
      - name: my-helloworld-app
        image: stacksimplify/kube-helloworld:1.0.0
```

_Deployment definition is similar, do check it out on your own_.


**NOTE**: Template field determines how a pod will be created (by workloads, such as ReplicaSets, deployments, DaemonSets, jobs, etc.). In this field you specify metadata and specification ("spec") values.

### Command to apply a YAML definition
We can apply the definitions in our YAML file using the command-line tool for Kubernetes kubectl, through the following command...

`kubectl apply -f <YAML file path>`

Of course, if your present working directory is the same as the one in which the YAML file is, the path is simply the file name.
