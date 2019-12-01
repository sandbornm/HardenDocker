<p align="center">
  <img width="800" height="300" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/banner.png">
</p>
                                        
<p align="center">
  <img width="460" height="420" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/hardendock.png">
</p>

# How to harden a Docker image: a tutorial for beginners

## This tutorial provides a basic overview of Docker and its security mechanisms, discusses best practices for creating Docker containers, and surveys a number of scanning and monitoring software to harden Docker images.

# Table of Contents
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

<p align="center">
  <img width="300" height="300" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/daemon.png">
</p>

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

<p align="center">
  <img width="360" height="120" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/scream.png">
</p>



**util.c** vulnerability [CVE-2018-9862]:

The util.c exploit was discovered in early 2018 and scored 7.8/10 (high) in the CVSS. This exploit leverages the mishandling of a numeric username which grants attackers root access when they use a specific value on a line in the `etc/passwd` file (text file containing attributes of each user, accessible to unprivileged users). While in a Docker container, an attacker can run `docker exec -u` and pass the numeric username value to obtian root access and impose its will on the compromised Docker container. (yikes again!)

![](https://github.com/sandbornm/HardenDocker/blob/master/assets/etcpasswd.png)

**Misconfigurations** in root accounts

On multiple occasions, containers have been found on DockerHub (think Github for Docker images) with root accounts that had blank passwords! This allows root access to modify the container to anyone who downloads the container from DockerHub since the password for `root` access is... nothing. Blank. There's **NO** password. Not good!! Misconfigurations like these are often overlooked and easily prevented by fortifying containers with strict role-based access control (RBAC). This means clearly specifying the access (read/write) privileges of each user on the image. 

**Lateral network movement**

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

### Docker Bench Security

Docker Bench Security is a very popular open source tool that allows a user to audit their Docker image to find vulnerabilities or violations of common best practices in creating and distributing Docker images. The script can be run directly inside of a Docker container using labels. The script provides output that gives feedback including `WARN`, `NOTE`, `INFO`, and `PASS` along with test descriptions to indicate what was being tested. At the end of this tutorial is a walkthrough to set up Docker Bench Security as it is a good (and free) starting point for hardening Docker images to check for any violations of best practices.

### (Another) Linux aside

Alpine Linux is a distribution of Linux known for its portability and security, making it a great pairing with Docker. For comparison, the minimal installation of Ubunutu (a popular distro of linux) requires just under 4GB of disk space. On the other hand, the minimal installation of Alpine requires right around 130MB. There are also security mechanisms such as stack smashing protection and Position Independent Executables (PIEs) in place to thwart large categories of vulnerabilities. Check out https://alpinelinux.org/about/ for more on Alpine Linux.

Let's continue to add to our defense toolkit by unpacking some scanning and monitoring software for your Docker image. In the next section, we'll take a look at how these scanners work and what they can tell you about what you are doing wrong (and right) with your Docker containers.

## Hardening Docker images

While best practices are great for the users and owners of a Docker image, sometimes they aren't enough. That's why there are software tools available that can tell you what you're doing wrong or failing to do that puts your Docker container at risk. These tools are called image scanners or monitors, and can't point you in the right direction on hardening your Docker image.

### How do container scanners work?

While it may not be absolutely necessary, it would be nice to understand how exactly scanning software is able to determine what is going well and what is not going so well in your Docker image.

Image scanning software works by parsing Docker image dependencies, i.e. the things in the Dockerfile and anything else in the image that the Dockerfile references, and determining whether any of the dependencies in their current versions have known vulnerabilities that can adversely affect the image being interrogated. Typically, these scanning tools are open source to benefit from the combined collaboration of the community, making the scanners better for everyone. In addition, different scanners have different approaches for identifying vulnerabilities and also for reporting them and providing feedback for the image owner.

It's important to note that no scanning software will identify every possible vulnerability in a Docker image. This is by virtue of the inherent complexity of software as well as the dynamic landscape of software vulnerabilities. In addition, scanners cannot check for an **unknown** vulnerability! Clearly, these scanners should not be the single line of defense in hardening your docker image! Rather, these scanners should be employed with best practices as well as due diligence in securing the host and resources associated with your Docker image. This is the best you can do to harden your Docker image, and sometimes even all of these measures might not be enough! There is no such thing, in current practice, as 100% secure software. But, you should strive for as close to it as possible!  

### Scanning tools and highlights

**AppArmor**

<p align="center">
  <img width="150" height="150" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/apparmor.png">
</p>

AppArmor is a Linux security module that comes built-in to Docker. AppArmor allows the system administrator to specify certain security profiles to be used when certain applications are run on the operating system. AppArmor can detect certain security threats and alert the appropriate user when a threat is detected on a specific profile. In Docker, the default security profile is called `docker-default` and has moderate security. When running a container, a certain security profile can be specified from AppArmor by overriding the `--security-opt` command line arg. Otherwise, the default profile is used to run the container. It would be wise to create and run a Docker container with at least one AppArmor security profile that provides greater security than the `docker-default` profile.

**Dagda**

<p align="center">
  <img width="200" height="150" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/dagda.png">
</p>

Dagda is a powerful open source tool to scan your Docker image for vulnerabilities, trojans, viruses, and malware. Dagda can also monitor the Docker daemon as well as running containers to keep an eye out for suspicious activity. This tool works by aggregating known vulnerabilities from multiple sources into a noSQL database and then checking the database as the image is scanned for bad dependencies, versions, or languages. Dagda also uses antivirus APIs to check for dormant malware in Docker images. Dagda supports multiple operating systems and languages. 

**StackRox**

<p align="center">
  <img width="150" height="150" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/stackrox.png">
</p>

Stackrox is a paid, enterprise level container security manager for multiple environments, including Docker. StackRox boasts full lifecycle security of Docker containers and allows for seamless integration with Docker engine and DockerHub. StackRox has the ability to block Docker images with vulnerabilities from being deployed and also supports third-party scanners. This software also provides a network map to monitor how containers are interacting and watches for suspicious activity and can also provide assessments for security standard benchmarks. Finally, StackRox provides runtime protection from threats with automated policy enforcement to take down any images that don't adhere to defined standards or rule whitelists. This is a heavy-duty product for Docker containers at enterprise scale.

**Anchore**

<p align="center">
  <img width="150" height="150" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/anchore.png">
</p>

Anchore is an open source project that advertises quick and precise scanning of container images to determine vulnerabilities with easy installation. The scanner itself is packaged as a Docker image to be run on the Docker image under inspection. Anchore advertises its solution as the most comprehensive security inspection platform and checks for known vulnerabilities, exposed credentials, operatinhg system packages, 3rd party libraries, whitelist allowed elements, blacklist sensitive elements, analyzes Dockerfile contents, and identifies config files, file permissions, and unpackaged files. Here is a straighforward walkthrough to try it yourself: https://anchore.com/docker-image-security-in-5-minutes-or-less/

**Clair**

<p align="center">
  <img width="240" height="80" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/clair.jpg">
</p>

Clair is yet another open source Docker image scanner that aggregates vulnerability data from multiple sources and cross-references them with those scanned in a Docker image. The result of a scan is a list of vulnerabilities that threaten the container. Clair also provides the capability to notify affected containers when upstream vulnerability data changes, and to create responses to such changes programmatically as a response to maintain the security of the Docker image. 

**Tenable** 

<p align="center">
  <img width="240" height="80" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/tenable.png">
</p>

Tenable is another paid, enterprise level scanner that advertises its ability to eliminate vulnerabilties early via end-to-end visibility of Docker images with vulnerability assessment, malware detection, and policy enforcement. Highlight features include: automated inspection of each layer of an image, continuous assessment of threats by updating vulnerability data, policy assurance detection via risk threshold violations, runtime security and notification of container modification. Similar to StackRox, Tenable also has an appealing visualization dashboard providing metrics on an image or cluster of images.

Now, on to *monitoring* software... 

### How do container monitors work?

Container monitors work similar to scanners except they work while a Docker image is up and running, i.e. a container (remember?!). While scanners check the contents of an image including its Dockerfile and other dependencies and contents of other locations in the image, a monitor must keep track of everything that happens while a container is interacting with another entity (an orchestrator, another container, multiple containers, etc.). Keeping track of metrics while a container is working and interacting allows a monitor to determine when a container is performing as expected, and when something has gone wrong. In the latter case, good monitoring tools must be able to determine exactly what went wrong and how to fix it in as little time as possible. Monitoring is essential to detect problems in containers as early as possible to mitigate backlash at scale, to conduct updates under a close eye, and to improve container performance by analyzing metrics.

Compared to image scanners, Docker monitors can be considered "active scanners" since they observe Docker containers that are up and running. This doesn't mean monitoring tools are better than scanners, as the same shortcomings as scanners are inevitable.

### Monitoring tools and highlights

**cAdvisor**

<p align="center">
  <img width="100" height="100" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/cadvisor.png">
</p>

cAdvisor is an open source monitoring tool for Docker backed by Google. The entire tool is a single Docker container that can be run from within the container to monitor and has its own GUI providing statistics of the container in which it's run. cAdvisor can collect and process relevant information from the running containers to determine if anything is amiss in your Docker image. cAdvisor connects directly to the Docker daemon and begins checking resource consumption like CPU and memory usage and Network I/O to detect any suspicious activity. cAdvisor works best with a single Docker host.

**Scout**

<p align="center">
  <img width="100" height="100" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/scout.png">
</p>

Scout is a paid monitoring tool that costs $10 per host. Scout works well with multiple host and maintains data for a long period of time. Similar to cAdvisor, Scout reports metrics related to resource usage, checks for bottlenecks in job queues, and checks performance across multiple Docker deployments. Scout also provides an intuitive UI that makes it easy for the administrator to identify and resolve issues affecting performance and compromised security.

**Datadog**

<p align="center">
  <img width="100" height="100" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/datado.jpg">
</p>

Datadog is a full-stack, cloud-based monitoring service that costs $15 per host. Datadog can monitor container environments with ease. Compared with cAdvisor and Scout, Datadog emphasizes very specific metrics of Docker container deployments across multiple hosts which makes it very easy to identify issues and closely monitor running applications. A unique feature of Datadog is to flag specific issues that may come up and automate a response to them for future occurrences, including an alert that the issue has repeated. 

**Sensu**

<p align="center">
  <img width="225" height="225" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/sensu.png">
</p>

Sensu is a monitoring system with its own API container on DockerHub that connects with the Sensu server to run as a single container on the host machine. Sensu requires a set of plugins to play nicely with Docker container metrics and status. Sensu also relies on external appplications to launch processes that Sensu needs to work well. These include things such as Sensu API, Sensu Core, Redis, and Graphite. Likely because of all the hassle complexity, Sensu is a free monitoring tool that is highly configurable at the cost of considerable overhead.

**Sematext**

<p align="center">
  <img width="225" height="225" src="https://github.com/sandbornm/HardenDocker/blob/master/assets/sematext.jpg">
</p>

Sematext is a monitor and logger that collects and analyzes data on application performance. Sematext runs as a single container on each monitored host and logs data for each container independently. Sematext boasts compatibility with a handful of Docker tools, including Docker Swarm, Docker cloud, Docker datacenter, Amazon EC2, Kubernetes, Mesos, and Google containers. Sematext, like competitors, also has an intuitive UI to track many different metrics for the monitored containers to ensure anomalies are identified and resolved to minimize downtime.

### A quick note on some industry Docker applications



### Summary 

Docker is a very powerful software that takes advantage of clever features of the Linux kernel. Docker allows a developer to package applications into lightweight containers for fast and easy deployment. At scale, many containers may be running at any given time working together to complete a task or provide a service. For personal use, Docker may come in handy for an application or a software project. In any case, security should be a top priority. Providing a high level of security to a Docker image is no small task, and hardening a Docker image entails ensuring best practices and scanning relevant components of the image to make sure there are no footholds for attackers to cling to and compromise your Docker image. My hope is this tutorial has sufficiently described the actions that must be taken to ensure your Docker image is as secure as possible, and that you have learned something about containers and software security in general. Use the following checklist if you ever find yourself wondering if you forgot to check or do anything on the quest to secure your Docker image. Happy Hardening! :beers:

### Checklist

- [ ] Docker up-to-date?
- [ ] Content Trust enabled?
- [ ] AppArmor non-default security profile enabled?
- [ ] Minimal privilege user created to build image?
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
12. https://docs.docker.com/engine/security/apparmor/  
13. https://www.projectatomic.io/docs/docker-and-selinux/  
14. https://code-maze.com/top-docker-monitoring-tools/  
