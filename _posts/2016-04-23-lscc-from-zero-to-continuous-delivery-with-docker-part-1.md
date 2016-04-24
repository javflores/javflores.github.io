---
layout: post
title:  From zero to Continuous Delivery with Docker, day 1
---

Sometimes I use my blog as a place to store my notes online.
Today I attended the first day of the Workshop **From zero to Continuous Delivery with Docker**, carried out by Robert Firek and some of his colleagues from Codurance, a special event for LSCC.
I have tried to learn Docker a few times, ranging from an hour talk to a full Pluralsight video, I still didn't feel confident enough to apply it in my job.

Still one more day left to finish, but this two day workshop seems to be the final push I needed with this amazing technology that is Docker.
I know there are plenty of blogs out there talking about Docker, but here are my notes:

## Introduction to Docker

In the early beginning of the Internet, web servers run in a bare metal physical machine. Everything was fine until people started to think: *why don't we run several apps in the same machine*.
Problems started to appear, compatibility issues between different runtimes, environments, also the cost was really high.
First solution: use virtual machines running in those bare metal machine. Some other problems appeared
Problem: it is very demanding for the physical machine, since a vm runs a whole operating system.

In Linux there is something called **containers**, which are a lot simpler.
They are based on standard Linux technologies:

- **namespaces**: an application will run in a namespace.
- **cgroups**: it gives access to hardware, it gives some % of resources.
- **unionfs**: they are changeable file system.

All this is already in Linux but Docker put it all together and hides this for us into something called libcontainer.

A Docker image is based on layers. Each layer will add the required files to the unionfs. One layer can be linux, java,..
When we run a container, Docker will pull image, it creates the container, prepares the filesystem, network and finally it runs the application.

Somethings we can do with containers:

1. Pull: gets an image from a registry
2. Run: creates a container.
3. Build: runs the container
4. Push: push an image to the registry.

## Docker commands

We need a machine running a Docker daemon, Linux, Mac or Windows.

### Basic commands

Get the version of Docker:

{% highlight cs %} 
docker version
{% endhighlight %}

