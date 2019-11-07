# A security tutorial for Docker *beginners* - How to harden your Docker image

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
The common thread in every Docker image is the **Dockerfile**. This file is nothing more than a text document that provides commands and configuration inctructions for loading and installing relevant programs, tools, and files needed to successfully build and run the Docker image. These items are known as **dependencies**. Think of the Dockerfile like a chocolate chip cookie recipe: all of the necessary programs and commands (dependencies!) are the ingredients, and they are executed in order from top to bottom according to the instructions on each line of the Dockerfile. When all of these steps are performed using all of these ingredients, the end result is a ~~fresh batch of chocolate chip cookies~~ Docker image! Here's an example of a Dockerfile:

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

### Default security mechanisms
1. namespaces provide the layer of isolation between the Docker container and the Linux kernel. A **namespace** specifies kernel resources to which a set of processes has access. A Docker container has a set of namespaces to provide isolation between other containers and the rest of the kernel. This narrows the permissions of a container in several different places:

* PID (process ID) namespace to isolate processes
* NET (network) namespace to control network interfaces
* IPC (interprocess communication) to manage access to shared resources between processes
* MNT (mount) namespace to manage filesystem mount points (a **filesystem** controls how data is stored and retrieved, a **mount point** is a directory to access files and folders on disk)
* UTS (Unix time sharing) namespace isolates the data that identifies the kernel and version

All of these namespaces work together to basically ensure that there is mutual respect between the kernel and the container, and they strictly define what is shared between the two entities.

2. control groups

3. UnionFS 

4. network interface

### daemons and clients and engines ~~(Oh my!)~~
docker daemon
docker client
docker engine

### Structure of a Docker containers
Well isolated by default- ability to control level of isolation from network, storage, or other subsystems from other images and/or host machine. 

containers are generally small not many entry points

## Understanding how Docker containers are compromised

### Common error patterns

### HUMANS

### Exploits

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

### Some jargon

### Insecure vs. Secure containers

## Appendix

### Sources
https://resources.whitesourcesoftware.com/blog-whitesource/container-security-scanning
https://geekflare.com/docker-architecture/
https://docs.docker.com/engine/security/security/
https://sysdig.com/blog/docker-image-scanning/
https://www.redhat.com/en/topics/linux/what-is-the-linux-kernel
https://medium.com/@nagarwal/understanding-the-docker-internals-7ccb052ce9fe

### Additional info

### How to set up a Docker sandbox


