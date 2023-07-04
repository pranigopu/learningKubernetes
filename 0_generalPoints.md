# General points
## General computing concepts
### Platform
A machine (physical or virtual) on which software can be installed or run. The term "machine" can refer to a computer or a hardware device (maybe combined with an operating system) or a virtual environment (such as a virtual machine).

### Software system
A set of intercommunicating software-based components forming part of a platform.

### Computing environment
The computing environment is the collection of hardware, software and network components that support the processing, storage and exchange of information demanded by the software solution.

> Reference: https://www.sciencedirect.com/topics/computer-science/computing-environmen


### Runtime
A version of a program that can be run but not changed (i.e. purely executable file).

### Mounting
Mounting is a process by which a computer's OS makes files and directories on a storage device (ex. hard drive, CD-ROM, network share, etc.) available to users via the computer's filesystem.

#### Mount point & unmounting
In general, the process of mounting comprises the OS

1. Acquiring access to the storage medium
2. Recognizing, reading and processing filesystem structure + metadata on it
3. Registering the above to the  virtual filesystem (VFS) component

The location in the VFS to which the newly mounted medium was registered is called a mount point. When the mounting process is completed, the user can access the storage medium's files and directories from the mount point.

Unmounting is the inverse process. Here, the OS

1. Cuts off all user access to files and directories on the mount point
2. Writes the remaining queue of user data to the storage device
3. Refreshes filesystem metadata
4. Relinquishes access to the device, making the storage device safe for removal

### Namespace
A namespace is a declarative (i.e. explicitly defined) region in a code/ that provides a scope to the identifiers (the names of types, functions, variables, etc.) inside it. Namespaces are used to organize code into logical groups and to prevent name collisions that can occur especially when your code base includes multiple libraries

> Reference: https://learn.microsoft.com/en-us/cpp/cpp/namespaces-cpp?view=msvc-170
