
```
  _    _                  _                _____                _                   
 | |  | |                | |              |  __ \              | |                  
 | |__| |  __ _  _ __  __| |  ___  _ __   | |  | |  ___    ___ | | __ ___  _ __ 
 |  __  | / _` || '__|/ _` | / _ \| '_ \  | |  | | / _ \  / __|| |/ // _ \| '__|    
 | |  | || (_| || |  | (_| ||  __/| | | | | |__| || (_) || (__ |   <|  __/| |       
 |_|  |_| \__,_||_|   \__,_| \___||_| |_| |_____/  \___/  \___||_|\_\\___||_|       
                                                                                
 
 
 ```
                                                               
<p align="center">
  <img width="460" height="420" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/hardendock.png">
</p>

# How to harden a Docker image: a tutorial for beginners

## Table of Contents
### [How does Docker work?](##how-does-docker-work)
### [Understanding default security configurations](##understanding-default-security-configurations)
### [How can the security configuration change?](##understanding-how-docker-containers-are-compromised)
### [Tools to audit and harden your Docker images](##tools-to-audit-and-harden-your-docker-images)
### [Interpreting feedback and some examples](##interpreting-feedback-and-some-examples)
### [Appendix](##appendix)

## How does Docker work?

### Introduction
So, you've decided to use Docker in your programming endeavors. However, you've noticed that the exsiting documentation for Docker is hopelessly esoteric for beginners. Luckily, you've come to the right place: this tutorial will outline Docker as an open-source software, but more importantly, it will discuss the reltaive security implications of Docker images and the steps necessary to audit and "harden" or secure them. A rough overview of Docker is in order to get started:

### What is Docker?
Docker is an open source software that surfacede in 2013 that allows users to build and distribute full applications in compact files called *images*. When these images are built and run on a machine, they become containers: isolated environments with their own resources to accomplish a task or provide a service.

### How does one create a Docker image?
The common piece of *every* Docker image is the **Dockerfile**. This file is nothing more than a text document with commands and configuration instructions for loading and installing programs, tools, and files needed to successfully build and run the Docker image. A command could be to install a package, make a new database, or pass a variable. These items are known as **dependencies**. Think of the Dockerfile like a chocolate chip cookie recipe: all of the necessary programs and commands (dependencies!) are the ingredients, and they are executed in order from top to bottom according to the instructions on each line of the Dockerfile. When all of these steps are performed using all of these ingredients, the end result is a ~~fresh batch of chocolate chip cookies~~ Docker image! Here's an example of a Dockerfile:

![](https://github.com/sandbornm/HardenDocker/blob/master/assets/An-example-of-dockerfile.png)

A Docker image that presently running on a machine is referred to as a **container** and is a running instance of a Docker image. For those of you familiar with object-oriented programming, think of the Docker image as a class and the Docker container as an instance of the class. Or, to continue the cookie analogy, the Docker container is the cookie baked according to all of the ingredients and procedures in the Dockerfile!

### Why should I care?
Docker is useful because it allows developers to create applications and package them in lightweight and independent containers that can run almost anywhere. The next thing to understand about Docker images is how they interact with the host machine: what can the Docker image access on the host machine? What can the host machine access on the Docker image? How do these things change when the **image** becomes a **container**?

## How does the host machine interact with your Docker container?

### A Linux aside
To understand how these interactions take place we will first unpack a few essential concepts that make *containerization* possible:

Linux :penguin: is an operating system like OSX or Windows and allows the user to run application and connect to networks, etc. The **Linux kernel** is a very important subset of the Linux operating system that serves as the bridge between the hardware and the processes of a computer by managing and orchestrating the resources and tasks of the machine. The kernel has 4 main (very, very important) responsibilities:

1. **Memory Management** - where to store specific data and when to delete it
2. **Process Management** - organize when and how long certain processes can access the CPU
3. **Device drivers** - handle the interface between hardware and the processes
4. **System calls/security** - handle requests from processes to access other resources

The kernel has its own isolated memory space in the operating system to acomplish these tasks. An **operating system** supports a computer's basic responsibilities. This includes things like scheduling tasks, running applications, and controlling connected devices (like bluetooth keyboards). 

Think of the kernel as the staff at your favorite restaurant. Hosts/hostesses are responsible for determining which tables are empty and which need to be prepared for future guests (memory management). The host or hostess should also ensure that all parties are served by one or more waiters/waitresses (process management). The waiters/waitresses interpret what the guests order and communicate this information to the kitchen (device drivers). Finally, hosts or waiters can respond to requests from guests about their dining experience and are responsible for ensuring guests have a comfortable and satisfying visit to the restaurant (System calls and security). 

So what does the Linux kernel have to do with Docker?

Docker takes advantage of the Linux kernel in a special way to provide containerization to the user. The most important of these properties is the idea of a **shared kernel architecture**. This means that Docker containers *share* the kernel of the host operating system. This is part of what makes Docker containers so lightweight and efficient: they share the core resources with the machine that *hosts* the container. For security, there are a few mechanisms in place to ensure that a Docker container does not have unrestricted access to the kernel and the kernel does not have unrestricted access to the container.

![](https://github.com/sandbornm/HardenDocker/blob/master/assets/docker-and-kernel.jpg)

### Default Security mechanisms
#### Namespaces
1. namespaces provide the layer of isolation between the Docker container and the Linux kernel. A **namespace** specifies kernel resources to which a set of processes has access. In other words, a namespace limits what some group of processes can "see" in the kernel. If a group of processes can't "see" something, they can't use it. A namespace can be thought of as a *key* to a lock: a container can only access resources to which it has the namespace (key), and nothing else. Have you ever tried to open a lock without the key or with the incorrect key? (it doesn't (*shouldn't*) work!). The set of namespaces provides isolation between a given container, other containers, and the rest of the kernel, and gives a single container the illusion of having total access to the kernel (because it can't see what else has access to it). Namespaces exist for several areas of the kernel:

* PID (process ID) namespace: isolates processes
* NET (network) namespace: controls network interfaces
* IPC (interprocess communication) namespace: manages access to shared resources between processes
* MNT (mount) namespace: manages filesystem mount points (a **filesystem** controls how data is stored and retrieved and a **mount point** is a directory to access files and folders on disk)
* UTS (Unix time sharing) namespace: isolates the data that identifies the kernel and current version

All of these namespaces work together to ensure that there is "mutual respect" or boundaries between the kernel and the container, and they strictly define the resources that are shared between the two entities.

#### Control groups (cgroups)
2. Control groups provide a layer of access control to the Docker container to access the host operating system and vice versa. A **control group** or **cgroup** allows resources like CPU time, memory, and network access to be allocated across running processes. Cgroups can be configured, monitored, and modified to change *how much* of a certain resource can be used by a container. A control group can be thought of as an *accountant* for the kernel's resources: the control group must keep track of how the resources are being used and allocated, ensure that containers are not promised more  than the available resources, and ensure that no one container consumes more than its allocated share of the kernel resources. 

#### Network interfaces
3. Docker containers each have their own network interface, which means they don't have access to the network interfaces (ports, addresses, etc.) of other containers. Containers can interact via their network interfaces only after exchanging permission through the host. Once this is accomplished, containers can send packets (pieces of information) and establish connections with other containers or applications using available ports and/or socket connections.

#### Secure Computing Mode (Seccomp)
4. This is a Linux kernel feature that allows the administrator to restrict the actions available within a container. Restricting the actions that a container can take on the host system reduces the risk of having a host machine compromised by an infected Docker container.

These security mechanisms all work together to ensure that containers have appropriate  access and visibility to kernel resources, and that a given container plays nicely with other containers (if any) as well as the host machine.

To coordinate the actions of a Docker container and to interact with a single container or multiple other containers, there are a few important Docker tools to understand before examining vulnerabilities and container hardening. We investigate these tools next.

### engines and daemons and clients ~~(Oh my!)~~
#### Docker engine
The Docker engine is simply an application with a client, a server, and an Application Programming Interface (API). In other words, the Docker engine has a client to make requests, a server that grants those requests (if possible) and an API to carry out those requests by possibly communicating with other Docker engines or containers. The Docker daemon listens for requests and is considered the server component of the engine, and the client provides an interface for users to make requests to be completed by the daemon. The API specifies programs and actions that are useful to direct the daemon on how to fulfill requests from the client. These 3 main components together make Docker possible. 

#### Docker daemon
First, let's take a look at what a **daemon** is: a daemon is nothing more than a computer process that runs in the background. What good does this do? It allows for a program to handle requests automatically by *listening* for them so a user does not have to manually handle them. Daemons are used widely in operating systems for tasks such as establishing internet connections and protocols, providing ssh connectivity, and scheduling jobs.

The Docker **daemon** simply listens for requests and also manages Docker images and containers. The Docker daemon can also coordinate with other Docker daemons to provide Docker services to a number of Docker containers. This facilitates scalability and consistency across multiple containers.

\\daemon image

#### Docker client
The Docker **client** communicates user input commands to the Docker daemon which then executes the commands to modify, connect, or interact with another Docker container. This is achieved with the Docker API. The client is also able to communicate with more than one Docker daemon, making it possible to complete multiple tasks on different containers using the same client. The client is involved any time a user enters a command prefaced with `docker` into a command line such as a terminal or shell. The client makes it possible to easily manipulate and interact with Docker containers after they are created.

One of the attractive things about containerization technology like Docker is that containers provide a layer of isolation from the host machine and operating system whcih provides additional security for the resources of the host and the container itself. These default features are made possible by the combination of *namespaces*, *cgoups*, and *network interfaces*.

Docker containers have many default settings and configurations out of the box that provide a reasonable level of security to both containers and hosts. Unfortunately, this is not always enough to protect Docker containers from compromise. In the next sections, we will examine how exactly Docker containers are compromised: from command line blunders to Dockerfile misconfigurations to large attack surfaces in the Docker image itself.

![](https://github.com/sandbornm/HardenDocker/blob/master/assets/overview.png)

## How are Docker containers compromised?

As with many software systems, humans tend to expose unnecessary information or include extraneous items in our systems that can ultimately lead to the demise of a system. Docker images are no different! Here is a totally not comprehensive list outlining a few ways Docker images are compromised: 

* Exposed credentials: this mishap leaves Docker containers vulnerable to attack via the entry point exposed by the credentials. This may include login information to a database, a server IP address/port, or an API key for an application.

* Failure to secure network privileges: this allows attackers to infiltrate a Docker container via its network stack (also: Docker containers don't play nicely with traditional firewalls which keep a list of rules in an `iptable` to discern malicious and friendly connections. This means it's **super** important to understand default network configurations on your Docker image!)

* Failure to adequately audit dependencies: using dependencies or images pulled from open source repositories guarantees nothing about the safety of your Docker containers! It's important to understand the implications of using certain dependencies and keeping them up-to-date and monitoring their vulnerabilities. Obsolete packages or services specified in the Dockerfile are susceptible to attackers exploiting deprecated dependenices and can lead to a crippled Docker container - or an infected host.

With these human mistakes in mind, we turn our attention in the next section to a few known Docker vulnerabilities that are facilitated by human error.

### Known vulnerabilities

**runC** vulnerability [CVE-2019-5736]:  

The runC exploit was discovered at the beginning of 2019 and scored 8.6/10 (high severity) in the CVSS (common vulnerability scoring system). This exploit leverages mishandling of file descriptors (an indicator to access an I/O medium like a file or socket) in `/proc/self/exe` (a file for handling running processes) in new or existing images. The runC is an executable that runs in background when Docker starts up to manage running containers. The file descriptor mishandling allows the executable to be overwritten with another executable specified by the attacker i.e. an attacker could execute arbitrary commands with root access. This type of command injection leaves the host machine and its resources in the hands of the attacker (yikes!)

![](https://github.com/sandbornm/HardenDocker/blob/master/assets/scream.png)

**util.c** vulnerability [CVE-2018-9862]:

The util.c exploit was discovered in early 2018 and scored 7.8/10 (high) in the CVSS. This exploit leverages the mishandling of a numeric username which grants attackers root access when they use a specific value on a line in the `etc/passwd` file (text file containing attributes of each user, accessible to unprivileged users). While in a Docker container, an attacker can run `docker exec -u` and pass the numeric username value to obtian root access and impose its will on the compromised Docker container. (yikes again!)

![](https://github.com/sandbornm/HardenDocker/blob/master/assets/etcpasswd.png)

**Misconfigurations** in root accounts

On multiple occasions, containers have been found on DockerHub (think Github for Docker images) with root accounts that had blank passwords! This allows root access to modify the container to anyone who downloads the container from DockerHub since the password for `root` access is... nothing. Blank. There's **NO** password. Not good!! Misconfigurations like these are often overlooked and easily prevented by fortifying containers with strict role-based access control (RBAC). This means clearly specifying the access (read/write) privileges of each user on the image. 

**Lateral network movement

Just like non-containerized software applications, Docker containers are also susceptible to compromise via lateral network movement. This is when an attacker assumes the identities of various users and attempts to compromise different portions of the Docker image. Different users may have different privileges, allowing the attacker to glean all kinds of information about the Docker image and eventually take over the whole container. This type of attack is especially scary since Docker containers are not protected by traditional firewalls (since they have their own network stack) and because a single Docker host (the machine running Docker) can attack any other host (another machine) on the same network! :scream:

We've seen the turmoil that Docker exploits can stir up, so let's take a look at measures we can take from a defensive standpoint to ensure our Docker containers are as secure as possible.

## Best practices for Docker

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

Let's continue to add to our defense toolkit by unpacking some scanning and monitoring software for your Docker image. In the next section, we'll take a look at how these scanners work and what they can tell you about what you are doing wrong (and right) with your Docker containers.

## Hardening Docker images

While best practices are great for the users and owners of a Docker image, sometimes they aren't enough. That's why there are software tools available that can tell you what you're doing wrong or failing to do that puts your Docker container at risk. These tools are called image scanners or monitors, and can't point you in the right direction on hardening your Docker image.

### How do container scanners work?

While it may not be absolutely necessary, it would be nice to understand how exactly scanning software is able to determine what is going well and what is not going so well in your Docker image.

Image scanning software works by parsing Docker image dependencies, i.e. the things in the Dockerfile and anything else in the image that the Dockerfile references, and determining whether any of the dependencies in their current versions have known vulnerabilities that can adversely affect the image being interrogated. Typically, these scanning tools are open source to benefit from the combined collaboration of the community, making the scanners better for everyone. In addition, different scanners have different approaches for identifying vulnerabilities and also for reporting them and providing feedback for the image owner.

It's important to note that no scanning software will identify every possible vulnerability in a Docker image. This is by virtue of the inherent complexity of software as well as the dynamic landscape of software vulnerabilities. These scanners should not be the single line of defense in hardening your docker image! Rather, these scanners should be employed with best practices as well as due diligence in securing the host and resources associated with your Docker image. This is the best you can do to harden your Docker image, and sometimes even all of these measures might not be enough! There is no such thing, in current practice, as 100% secure software. But, you should strive for as close to it as possible!  

### Scanning tools and highlights

apparmor - linux security module 
SElinux - open source 
docker bench security - open source
Stackrox
anchore - open source
clair - open source

### Monitoring tools and highlights
Scout
Datadog
Prometheus

### Bench Security Setup 

https://www.digitalocean.com/community/tutorials/how-to-audit-docker-host-security-with-docker-bench-for-security-on-ubuntu-16-04#step-1-%E2%80%94-installing-docker-bench-security

### Checklist

Refer to this checklist to harden your Docker image and prevent it from being pwned:

- [ ] Docker up-to-date?
- [ ] Content Trust enabled?
- [ ] Bench Security enabled?
- [ ] Low privilege user created?
- [ ] Use `COPY`over `ADD` unless absolutely necessary?
- [ ] Use `RUN` with `apt-get update && apt-get install`?
- [ ] Third party scan of Docker image with feedback?
- [ ] No exposed credentials in Dockerfile?

### Sources
1. https://resources.whitesourcesoftware.com/blog-whitesource/container-security-scanning  
2. https://geekflare.com/docker-architecture/  
3. https://docs.docker.com/engine/security/security/  
4. https://sysdig.com/blog/docker-image-scanning/  
5. https://www.redhat.com/en/topics/linux/what-is-the-linux-kernel  
6. https://medium.com/@nagarwal/understanding-the-docker-internals-7ccb052ce9fe  
7. https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/ch01#sec-How_Control_Groups_Are_Organized  
8. https://medium.com/intive-developers/hardening-docker-quick-tips-54ca9c283964  
9. https://blog.aquasec.com/docker-security-best-practices  
10. https://www.docker.com/sites/default/files/WP_IntrotoContainerSecurity_08.19.2016.pdf  
11. https://resources.whitesourcesoftware.com/blog-whitesource/docker-image-security-scanning
