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

More info about the Docker daemon:

{% highlight cs %}
docker info
{% endhighlight %}

Help is really useful:

{% highlight cs %}
docker help
{% endhighlight %}

Example: docker help run

Find what images we've got downloaded:

{% highlight cs %}
docker images
{% endhighlight %} 

Images are stored in registries, the registry by default is DockerHub. To get an image into our machine:

{% highlight cs %}
docker pull ubuntu
{% endhighlight %}

That is the same as:
{% highlight cs %}
docker pull library/ubuntu
{% endhighlight %}

In general the convention in the registry is ```[username/]repository:tag```

We can navigate to Docker hub and [search for Linux Centos](https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=centos&starCount=0) and pull one of them.

If we want to have several images of the same thing like ubuntu you can put a tags. For example: ```docker pull library/ubuntu:14.04```. Suggestion, always use tags.

We can remove images (It is a dangerous operation):

{% highlight cs %}
docker rmi ubuntu:14.04
{% endhighlight %}

And with this:

{% highlight cs %}
docker rm $(docker ps -aq)
{% endhighlight %}

To run a command:

{% highlight cs %}
docker run busybox sh -c "echo Hello World"
{% endhighlight %}

In this case it pulled an image from DockerHub called busybox and run inside the container a shell, Hello world.

We can run a command based on name or label:

{% highlight cs %}
docker run --name juan busybox sh -c "echo Hello World"
docker run --label juan busybox sh -c "echo Hello World"
{% endhighlight %}

Some commands require to use the imageId, but we don't need to provide the full imageId, but the first two numbers or 3 numbers.

To get info about an image:

{% highlight cs %}
docker inspect <image-id>
{% endhighlight %}

And to see the logs:

{% highlight cs %}
docker logs <image-id>
{% endhighlight %}

Actually with the docker inspect command we can find where the log files of the process are and then we can see them with: 

{% highlight cs %}
sudo cat <file-path>
{% endhighlight %}

Let's run a more complicated program that prints Hello world in a while loop:

{% highlight cs %}
docker run -d --name loop_ubuntu ubuntu /bin/sh -c  "while true; do echo hello world; sleep 1; done"
{% endhighlight %}

We can kill it:

{% highlight cs %}
docker kill <id>
{% endhighlight %}

Or more gracefully:

{% highlight cs %}
docker stop <id>
{% endhighlight %}

Let actually run an application, we can use one of the training apps provided by Docker, in this case Python:

{% highlight cs %}
docker run -d --name web training/webapp python app.py
{% endhighlight %}

With -P it will expose a port number externally:

{% highlight cs %}
docker run -d -P training/webapp python app.py
{% endhighlight %}

To map a external port to an internal one:

{% highlight cs %}
docker run -d -p 8080:5000 -p 8081:5001 training/webapp python app.py
{% endhighlight %}

Let's see what containers we've got running:

{% highlight cs %}
docker ps -a -q
{% endhighlight %}

We can use this command to delete the running containers:

{% highlight cs %}
docker ps -a -q | xargs docker rm -f
{% endhighlight %}

To run a container interactively:

{% highlight cs %}
docker run -ti
{% endhighlight %}

For example: 

{% highlight cs %}
docker run -ti busybox sh
{% endhighlight %}

Now you are in the container and you can run the terminal. You can ```echo "Hello world" > HelloWorld.txt```
We can also:

{% highlight cs %}
mkdir my_busybox
cd my_busybox
nano Dockerfile
{% endhighlight %}

This way we have created a Dockerfile from inside the container. The Docker file can execute linux commands for us and much more.

Given a folder with a DockerFile we can build a container:

{% highlight cs %}
docker build
{% endhighlight %}

It is untagged so we can:

{% highlight cs %}
docker tag <tag
{% endhighlight %}

The following sets a new image based on that container or add changes to an image, container.

{% highlight cs %}
docker commit -m "My Busy Box" -a "Juan Vicaria" 42 my_busybox:v5
{% endhighlight %}

(we found 42 the image id of one container with docker ps -a)

Then we can run the container, even providing a parameter into the Docker run:

{% highlight cs %}
docker run my_bussybox:v5 -c "echo Nothing"
{% endhighlight %}



