# A tutorial on Docker and Docker security for *beginners*: How to harden your Docker image

## Assumptions: readers already have Docker Desktop and Docker CLI tools installed but are not familiar with containerization, how Docker works, or container security.

## Table of Contents
### [How does Docker work and what is an image?](##how-does-docker-work-and-what-is-an-image)
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

![](https://github.com/sandbornm/HardenDocker/blob/master/assets/docker-and-kernel.jpg)

### Default Security mechanisms
#### Namespaces
1. namespaces provide the layer of isolation between the Docker container and the Linux kernel. A **namespace** specifies kernel resources to which a set of processes has access. In other words, a namespace limits what some group of processes can "see" in the kernel. If a group of processes can't "see" something, they can't use it. A namespace can be thought of as a *key*: a container can only access resources to which it has the key, or namespace, and nothing else. The set of namespaces provides isolation between a given container, other containers, and the rest of the kernel and gives a single container the illusion of having total access to the kernel (because it can't see what else has access to it). Namespaces exist for several areas of the kernel:

* PID (process ID) namespace: isolates processes
* NET (network) namespace: controls network interfaces
* IPC (interprocess communication) namespace: manages access to shared resources between processes
* MNT (mount) namespace: manages filesystem mount points (a **filesystem** controls how data is stored and retrieved, a **mount point** is a directory to access files and folders on disk)
* UTS (Unix time sharing) namespace: isolates the data that identifies the kernel and version

All of these namespaces work together to ensure that there is mutual respect or boundaries between the kernel and the container, and they strictly define the resources that are shared between the two entities.

#### Control groups (cgroups)
2. Control groups provide a layer of access control to the Docker container to access the host operating system and vice versa. A **control group** or **cgroup** allows resources like CPU time, memory, and network access to be allocated across running processes. Cgroups can be configured, monitored, and modified to change *how much* of a certain resource can be used by a container. A control group can be thought of as an *accountant* for the kernel's resources: the control group must keep track of how the resources are being used and allocated, ensure that containers are not promised more  than the available resources, and ensure that no one container hogs all the kernel resources. 

reduce attack surface by restricting access to physical devices on the host- no access by default

#### Network interfaces
3. Docker containers each have their own network interface which means they don't have access to the network interfaces (ports, addresses, etc.) of other containers. Containers can interact via their network interfaces only after exchanging permission through the host. Once this is accomplished, containers can send packets (pieces of information) and establish connections with other containers or applications.

#### Secure Computing Mode (Seccomp)
4. This is a Linux kernel feature that allows the administrator to restrict the actions available within a container- restricts the actions that a container can take on the host system. 

These security mechanisms all work together to ensure that containers have appropriate kernel resource access and visibility, and that a given container plays nicely with other containers (if any) and the host machine.

To coordinate the actions of a Docker container and to interact with a single container or multiple other containers, there are a several important tools to understand before examining Docker vulnerabilities and container hardening...

### engines and daemons and clients ~~(Oh my!)~~
#### Docker engine
The Docker engine is simply an application with a client, a server, and an Application Programming Interface (API). In other words, the Docker engine has a client to make requests, a server that grants those requests (if possible) and an API to carry out those requests by possibly communicating with other Docker engines or containers. The Docker daemon listens for requests and is considered the server component of the engine, and the client provides an interface for users to make requests to be completed by the daemon. The API specifies programs and actions that are useful to direct the daemon on how to fulfill requests from the client. These 3 main components make Docker possible. 

#### Docker daemon
First, let's take a look at what a **daemon** is: a daemon is nothing more than a computer process that runs in the background. What good does this do? It allows for a program to handle requests automatically by *listening* for them so a user does not have to manually handle them. Daemons are used widely in operating systems for establishing internet connections and protocols, providing ssh connectivity, and scheduling tasks.

The Docker **daemon** simply listens for requests and also manages Docker images and containers. The Docker daemon can also coordinate with other Docker daemons to provide Docker services to a number of Docker containers. This facilitates scalability and consistency across multiple containers.

#### Docker client
The Docker **client** communicates user input commands to the Docker daemon which then executes the commands to modify, connect, or interact with another Docker container. This is achieved with the Docker API. The client is also able to communicate with more than one Docker daemon, making it possible to complete multiple tasks on different containers using the same client. The client is involved any time a user enters a command prefaced with `docker` into a command line such as a terminal or shell. The client makes it possible to easily manipulate and interact with Docker containers after they are created.

One of the attractive things about containerization technology like Docker is that containers provide a layer of isolation from the host machine and operating system whcih provides an additional security for the resources of the host and the container itself. These default features are made possible by the combination of *namespaces*, *crgoups*, and *network interfaces*.

As we have seen, Docker containers have many default settings and configurations out of the box that provide a reasonable level of security to containers and hosts- but this is not always enough. In the next sections, we will examine how exactly Docker containers are compromised- from the command line to misconfigurations to large entry points in the Docker image itself.

## How are Docker containers compromised?

As with many software systems, us humans tend to expose unnecessary information or include extraneous items in our systems that can ultimately lead to the compromise of a system to some extent. Docker images are no different!

Exposed credentials leave Docker containers vulnerable to attack via the entry point exposed by the credentials

Obsolete packages or services specified in the Dockerfile are susceptible to attackers exploiting deprecated dependenices

Failure to secure network privileges allows attackers to infiltrate a Docker container via its network stack (also: Docker containers don't play nicely with traditional firewalls which keep a list of rules in an `iptable` to discern malicious and friendly connections. This means it's that much more important to understand default network configurations on your Docker image!)

With these human mistakes in mind, we turn our attention to fundamental vulnerabilities in Docker containers that can be facilitated by human error.

### Common vulnerabilities

**runC** vulnerability [CVE-2019-5736]:

**util.c** vulnerability [CVE-2018-9862]:

vulnerabilites in container images
injecting with root access
lateral network movement 
exposure to insecure networks
exposure of hardware resources

We've seen the turmoil that Docker exploits can stir up, so let's take a look at measures we can take from a defensive standpoint to ensure our Docker containers are as secure as possible...

## Best practices for your Docker Images

### Dockerfiles

**`COPY` vs `ADD`**

The `COPY`instruction takes only 2 parameters: a `src` and a `destination`. In other words, only files that are in a local file or directory on the *host* system can be placed into the Docker image.

On the other hand, `ADD` allows the same functionality as `COPY`, with the *additional* capability of a remote URL and also extracting a `.tar` file directly into the Docker image. For obvious reasons, we don't want this capability near our Dockerfile, as it increases the attack surface of our image.

**`RUN`**

Always combine the `RUN`instruction with `apt-get update` and `apt-get install` to ensure the latest updates and packages are installed.

Only install *verified* and *necessary* packages: you should always be asking: "What is the bare minimum that I need for my Docker container to run as expected?" Include no more than needed in the Dockerfile!

Related to this- like classes in Object-Oriented Programming, Docker images should be decoupled from one another- best practice is to use Docker containers for small services to facilitate reuse and interdependency. 

### Avoid `root` access

Avoiding `root` access in your Docker container minimizes the chances of an attacker accessing host resources through the Docker container. To avoid `root` access in your Docker container, create a **low-privilege** user for then the Docker container is running. In the Dockerfile, create a low-privilege user with the following commands:

`RUN adduser -D low_privilege_user`  // add this user  
`USER low_privilege_user`            // use this user in container

Then when running your Docker container, specify this user:

`docker run -u low_privilege_user`

Use such a user to minimize access to host resources.

### Content trust

Docker Content Trust (DCT) allows a user of a Docker container to verify the integrity of incoming or outgoing data or the true author of a given docker image. This is accomplished with digital signatures which uses math to make a special string that represents a piece of raw data (including large files). Setting up DCT is a great way to assert the origin of certain assets for your Docker containers.

### (Another) Linux aside

Alpine Linux is a distribution of Linux known for its portability and security, making it a great pairing with Docker. For comparison, the minimal installation of Ubunutu (a popular distro of linux) requires just under 4GB of disk space. On the other hand, the minimal installation of Alpine requires right around 130MB. There are also security mechanisms such as stack smashing protection and Position Independent Executables (PIEs) in place to thwart large categories of vulnerabilities. Check out https://alpinelinux.org/about/ for more on Alpine Linux.

Let's continue to add to our defense toolkit by unpacking some scanning and monitoring software for your Docker containers...

## Hardening your Docker images 

### How do container scanners work?



### Scanning

apparmor
SElinux
docker bench security
Stackrox

### Monitoring 
Scout, Datadog, Prometheus

### Bench Security Setup 

https://www.digitalocean.com/community/tutorials/how-to-audit-docker-host-security-with-docker-bench-for-security-on-ubuntu-16-04#step-1-%E2%80%94-installing-docker-bench-security

### Sources
https://resources.whitesourcesoftware.com/blog-whitesource/container-security-scanning
https://geekflare.com/docker-architecture/
https://docs.docker.com/engine/security/security/
https://sysdig.com/blog/docker-image-scanning/
https://www.redhat.com/en/topics/linux/what-is-the-linux-kernel
https://medium.com/@nagarwal/understanding-the-docker-internals-7ccb052ce9fe
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/ch01#sec-How_Control_Groups_Are_Organized

https://medium.com/intive-developers/hardening-docker-quick-tips-54ca9c283964
https://blog.aquasec.com/docker-security-best-practices
https://www.docker.com/sites/default/files/WP_IntrotoContainerSecurity_08.19.2016.pdf
