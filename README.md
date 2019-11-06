# A security tutorial for Docker *beginners* - How to harden your Docker image

## Table of Contents
### [How does Docker work and what is an image?](##how-does-docker-work-and-what-is-an-image?)
### [Default security and host interactions](##default-security-and-host-interactions)
### [Understanding default security configurations](##understanding-default-security-configurations)
### [How can the security configuration change?](##how-can-the-security-configuration-change?)
### [Tools to audit and harden your Docker images](##tools-to-audit-and-harden-your-docker-images)
### [Interpreting feedback and some examples](##interpreting-feedback-and-some-examples)
### [Appendix](##appendix)

## How does Docker work and what is an image?

So, you've decided to use Docker in your programming endeavors. However, you've noticed that the exsiting documentation for Docker is hopelessly esoteric for beginners. Luckily, you've come to the right place: this tutorial will outline Docker as an open-source software, but more importantly, it will discuss the reltaive security implications of Docker images and the steps necessary to audit and "harden" or secure them. A rough overview of Docker is in order to get started:

Docker is an open source software that surfacede in 2013 that allows users to build and distribute full applications in compact files called *images*. When these images are built and run on a machine, they become containers: isolated environments with their own resources to accomplish a task or provide a service.

The common thread in every Docker image is the **Dockerfile**. This file is nothing more than a text document that provides instructions (commands) for installing relevant programs and tools needed to be successfully build and run the Docker image. Think of the Dockerfile like a recipe: all of the necessary programs and commands are the ingredients, and they are executed in order from top to bottom according to the instructions on each  line of the file. When all of these ingredients are used and all of the steps are performed, the end result is a ~~fresh batch of chocolate chip cookies~~ Docker image! Here's an example of a Dockerfile (recipe):

(example of Dockerfile && chocolate chip cookie recipe for fun)

A Docker image that is currently running is called a **container** and is referred to as a running instance of a Docker image. 

y

## Default security and host interactions



## Understanding default security configurations

## How can the security configuration change?

## Tools to audit and harden your Docker images

## Interpreting feedback and some examples

## Appendix

### Sources 

### Additional info

### How to set up a Docker sandbox
