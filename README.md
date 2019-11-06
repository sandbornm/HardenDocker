# A security tutorial for Docker *beginners* - How to harden your Docker image

## Table of Contents
### [How does Docker work and what is an image?](##how-does-docker-work-and-what-is-an-image?)
### [How does the host machine interact with your Docker container?](##how-does-the-host-machine-interact-with-your-docker-container?)
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

shared kernel architecture
take a detour into Linux
Linux kernel
namespaces
control groups
network interface

### daemons and clients and registries ~~(Oh my!)~~
docker daemon
docker client
docker registry

### Host interaction and privileged access

## Understanding default security configurations

### On the structure of Docker containers
Well isolated by default- ability to control level of isolation from network, storage, or other subsystems from other images and/or host machine. 

containers are generally small not many entry points



## Understanding how Docker containers are compromised

### Common error patterns

### HUMANS

### Exploits

attack surface of the docker daemon
runC root access remote execution

## Tools to audit and harden your Docker images

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

### Additional info

### How to set up a Docker sandbox


