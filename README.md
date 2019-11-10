# A tutorial on Docker and Docker security for *beginners*: How to harden your Docker image

## Assumptions: readers already have Docker Desktop and Docker CLI tools installed but are not familiar with containerization in general, how Docker works, or security practices for containers.

## Table of Contents
### [How does Docker work and what is an image?](##how-does-docker-work-and-what-is-an-image?)
### [Understanding default security configurations](##understanding-default-security-configurations)
### [How can the security configuration change?](##understanding-how-docker-containers-are-compromised)
### [Tools to audit and harden your Docker images](##tools-to-audit-and-harden-your-docker-images)
### [Interpreting feedback and some examples](##interpreting-feedback-and-some-examples)
### [Appendix](##appendix)

## How does Docker work and what is an image?

### Introduction
So, you've decided to use Docker in your programming endeavors. However, you've noticed that the exsiting documentation for Docker is hopelessly esoteric for beginners. Luckily, you've come to the right place: this tutorial will outline Docker as an open-source software, but more importantly, it will discuss the reltaive security implications of Docker images and the steps necessary to audit and "harden" or secure them. A rough overview of Docker is in order to get started:

### What is Docker?
Docker is an open source software that surfacede in 2013 that allows users to build and distribute full applications in compact files called *images*. When these images are built and run on a machine, they become containers: isolated environments with their own resources to accomplish a task or provide a service.

### How does this all work?
The common thread in every Docker image is the **Dockerfile**. This file is nothing more than a text document that provides commands and configuration inctructions for loading and installing relevant programs, tools, and files needed to successfully build and run the Docker image. A command could be to install a package, make a new database, or pass a variable. These items are known as **dependencies**. Think of the Dockerfile like a chocolate chip cookie recipe: all of the necessary programs and commands (dependencies!) are the ingredients, and they are executed in order from top to bottom according to the instructions on each line of the Dockerfile. When all of these steps are performed using all of these ingredients, the end result is a ~~fresh batch of chocolate chip cookies~~ Docker image! Here's an example of a Dockerfile:

![](https://github.com/sandbornm/HardenDocker/blob/master/assets/An-example-of-dockerfile.png)

A Docker image that is built and running on a machine is known as a **container** and is a running instance of a Docker image. For those of you familiar with object-oriented programming, we can think of the Docker image as a class and the Docker container as an instance of the class. To continue the cookie analogy, the Docker container is the cookie that was baked according to all of the ingredients and procedures in the Dockerfile!

### Why should I care?
Docker is useful because it allows developers and companies to create applications and package them in lightweight and independent containers that can be run almost anywhere. The next thing to understand about Docker images is how they interact with the host machine: what can the Docker image access on the host machine? What can the host machine access on the Docker image? How do these things change when the Docker image is built and run and becomes a Docker container?

## How does the host machine interact with your Docker container?

### A Linux aside
To understand how these interactions take place we will first unpack a few essential concepts that make *containerization* possible:

The **Linux kernel** is the bridge between the hardware and the processes of a computer and manages the resources of the device. The kernel has 4 main responsibilities:

1. Memory Management - where to store specific data and when to delete it
2. Process Management - organize when and how long certain processes can access the CPU
3. Device drivers - handle the interface between hardware and the processes
4. System calls/security - handle requests from processes to access other resources

The kernel has its own isolated memory space in the operating system to acomplish these tasks. An **operating system** supports a computer's basic responsibilities. This includes things like scheduling tasks, running applications, and controlling connected devices (like bluetooth keyboards). 

Think of the kernel as the staff at your favorite restaurant. Hosts/hostesses are responsible for determining which tables are empty and which need to be prepared for future guests (memory management). The host or hostess should also ensure that all parties are served by one or more waiters/waitresses (process management). The waiters/waitresses interpret what the guests order and communicate this information to the kitchen (device drivers). Finally, hosts or waiters can respond to requests from guests or parties about their dining experience and are responsible for ensuring guests have a comfortable and satisfying time at the restaurant (System calls and security). 

So what does the Linux kernel have to do with Docker?

Docker takes advantage of the Linux kernel in a special way to provide containerization to the user. The most important of these properties is the idea of a **shared kernel architecture**. This means that Docker containers *share* the kernel of the host operating system. This is part of what makes Docker containers so lightweight and efficient. They share the core resources with the machine that has the container. Luckily, there are a few mechanisms in place to ensure that a Docker container does not have unrestricted access to the kernel and the kernel does not have unrestricted access to the container.

### Security mechanisms
1. Namespaces provide the layer of isolation between the Docker container and the Linux kernel. A **namespace** specifies kernel resources to which a set of processes has access. In other words, a namespace limits what some group of processes can "see" in the kernel. If a group of processes can't "see" something, they can't use it. A namespace can be thought of as a *key*: a container can only access resources to which it has the key, or namespace, and nothing else. The set of namespaces provides isolation between a given container, other containers, and the rest of the kernel and gives a single container the illusion of having total access to the kernel (because it can't see what else has access to it). Namespaces exist for several areas of the kernel:

* PID (process ID) namespace: isolates processes
* NET (network) namespace: controls network interfaces
* IPC (interprocess communication) namespace: manages access to shared resources between processes
* MNT (mount) namespace: manages filesystem mount points (a **filesystem** controls how data is stored and retrieved, a **mount point** is a directory to access files and folders on disk)
* UTS (Unix time sharing) namespace: isolates the data that identifies the kernel and version

All of these namespaces work together to ensure that there is mutual respect or boundaries between the kernel and the container, and they strictly define the resources that are shared between the two entities.

2. Control groups provide a layer of access control to the Docker container to access the host operating system and vice versa. A **control group** or **cgroup** allows resources like CPU time, memory, and network access to be allocated across running processes. Cgroups can be configured, monitored, and modified to change *how much* of a certain resource can be used by a container. A control group can be thought of as an *accountant* for the kernel's resources: the control group must keep track of how the resources are being used and allocated, ensure that containers are not promised more  than the available resources, and ensure that no one container hogs all the kernel resources. 

4. Docker containers each have their own network interface which means they don't have access to the network interfaces (ports, addresses, etc.) of other containers. Containers can interact via their network interfaces only after exchanging permission through the host. Once this is accomplished, containers can send packets (pieces of information) and establish connections.

These security mechanisms all work together to ensure that containers have appropriate kernel resource access and visibility, and that a given container plays nicely with other containers (if any) and the host machine.

To coordinate the actions of a Docker container and to interact with a single container or multiple other containers, there are a few tools in place

### engines and daemons and clients ~~(Oh my!)~~
docker daemon
docker client
docker engine

### Structure of a Docker containers
Secure by default
Well isolated by default- ability to control level of isolation from network, storage, or other subsystems from other images and/or host machine. 

containers are generally small not many entry points

## How are Docker containers compromised?

### HUMANS

### Common vulnerabilities

### Known exploits

attack surface of the docker daemon
runC root access remote execution

## Docker security and tools to audit and harden your Docker images

### Best practices
Docker content trust
do not use root access
automate scanning of containers

### Automate, Automate, Automate

### Scanning software

## Interpreting feedback and some examples

### Insecure vs. Secure containers

## Appendix

### Some jargon

### Docker scanning resources
 
### Sources
https://resources.whitesourcesoftware.com/blog-whitesource/container-security-scanning
https://geekflare.com/docker-architecture/
https://docs.docker.com/engine/security/security/
https://sysdig.com/blog/docker-image-scanning/
https://www.redhat.com/en/topics/linux/what-is-the-linux-kernel
https://medium.com/@nagarwal/understanding-the-docker-internals-7ccb052ce9fe
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/ch01#sec-How_Control_Groups_Are_Organized

### How to set up a Docker sandbox


