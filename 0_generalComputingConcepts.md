# General computing concepts
## Logical vs. physical group
A logical set or group of entities is a group built by grouping references to the entities rather than the entities themselves. _Generally, a logical group is defined based on some relationship between the entities that unites them conceptually_. On the other hand, a physical set or group of entities is a group that groups the actual entities in a particular location.

## Computing environment
The computing environment of a software solution is the collection of hardware, software and network components that support the processing, storage and exchange of information demanded by the software solution.

> Reference: https://www.sciencedirect.com/topics/computer-science/computing-environment

## Platform
A machine (physical or virtual) on which software can be installed or run. The term "machine" can refer to a computer or a hardware device (maybe combined with an operating system) or a virtual environment (such as a virtual machine).

## Software system
A set of intercommunicating software-based components forming part of a platform.

## Runtime
A version of a program that can be run but not changed (i.e. a purely executable file).

## Mounting
Mounting is a process by which a computer's OS makes files and directories on a storage device (ex. hard drive, CD-ROM, network share, etc.) available to users via the computer's filesystem.

### Mount point & unmounting
In general, the process of mounting comprises the OS

1. Acquiring access to the storage medium
2. Recognizing, reading and processing filesystem structure + metadata on it
3. Registering the above to the virtual filesystem (VFS) component

The location in the VFS to which the newly mounted medium was registered is called a mount point. When the mounting process is completed, the user can access the storage medium's files and directories from the mount point.

Unmounting is the inverse process. Here, the OS

1. Cuts off all user access to files and directories on the mount point
2. Writes the remaining queue of user data to the storage device
3. Refreshes filesystem metadata
4. Relinquishes access to the device, making the storage device safe for removal

## Namespace
A namespace is a declarative (i.e. explicitly defined) region in a code/environment that provides a scope to the identifiers (the names of types, functions, variables, etc.) inside it. Namespaces are used to organize code into logical groups and to prevent name collisions that can occur especially when your code base includes multiple libraries

> Reference: https://learn.microsoft.com/en-us/cpp/cpp/namespaces-cpp?view=msvc-170

## Software agent
In computer science, a software agent or software AI (artificial intelligence) is a computer program that acts for (_i.e. on behalf of or with the software identity of_) a user or another program with some level of agency (i.e. delegated power and responsibility).

Examples of software agents:

- Monitoring and surveillance agent
- Data mining agent
- Shopping agent (buyer bot)

## Imperative vs. declarative programming paradigms
**_(Two of the most popular programming paradigms)_**

Imperative programming is the method of describing a step-by-step (i.e. ordered) process for a program's execution. In other words, imperative programming describes the control flow (_of the process derived from the program's execution_). **_Hence, emphasis is more on execution process_**. Types of imperative programming paradigms:

- Procedural programming paradigm
	- Ex. C, BASIC, Pascal
- Structured programming paradigm
- Object-oriented programming paradigm
	- Ex. C++, Java, Python

**NOTE**:<br>_Object-oriented langs ⊆  Structured langs ⊆ Procedural langs_<br>The wider category can support the paradigm of the narrower category.

Declarative programming is the method of describing the end result of the process. In other words, declarative programming describes the end result of a program's execution without describing its control flow. _It is up to the programming language's implementation and the compiler to determine how to achieve the results_. **_Hence, emphasis is more on the result of execution, and not the execution process itself_**. Types of declarative programming paradigms:

- Functional programming paradigm
	- Ex. JavaScript, R, Wolfram Language
- Logical programming paradigm
	- Ex. Prolog, SQL (to a limited degree)

**SIDE NOTE**: _A programming language may support various programming paradigms and not just one. For examples:_

- _Object-oriented languages support object-oriented, structured and procedural programming paradigms_
- _JavaScript supports both structured and functional programming paradigms_

## Task context
A task context is the minimal set of data used by a task (which may be a process, thread, or fiber) that must be saved to allow a task to be interrupted and later continued from the same point.

## Application programming interface (API)
### Endpoint
An API endpoint is a digital location where an API receives requests for a specific resource on its server. In practice, an endpoint is typically a uniform resource locator (URL) that provides the location of a resource on the server. In other words, an endpoint is the point of connection between the API and a specific resource.

**NOTE**: Software programs typically have multiple API endpoints. For example, Instagram's endpoints include

- An endpoint that allows businesses and creators to measure media and profile interactions
- An endpoint that allows them to moderate comments and their replies
- An endpoint that allows them to discover hash-tagged media

> References:
> - https://blog.hubspot.com/website/api-endpoint
> - https://www.techtarget.com/searchapparchitecture/definition/API-endpoint

## Proxy & reverse proxy
A proxy (also called forward proxy) is a component in a network that enables nodes in a private network to access a wider network. A reverse proxy is a component in a network that enables nodes outside a private subnet from accessing the private subnet's nodes. A load balancer is an example of a reverse proxy.
