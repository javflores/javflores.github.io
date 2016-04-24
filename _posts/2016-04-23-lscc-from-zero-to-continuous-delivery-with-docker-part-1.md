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

### Registries

We can create private registries to hold images. In the workshop we had a private one 

{% highlight cs %}
registry.training.local\
{% endhighlight %}

Before we tagged the image:

{% highlight cs %}
docker tag my_bussybox:v5 registry.training.local/juan_busybox
{% endhighlight %}

Then pushing the image given that my machine is connected to the private repo with the VPN:

{% highlight cs %}
docker push registry.training.local/juan_busybox
{% endhighlight %}

Finding all the images in a private repository:

{% highlight cs %}
curl registry.training.local/v2/_catalog | jq .
{% endhighlight %}

Finding all the history of an image and all the tags:

{% highlight cs %}
curl registry.training.local/v2/juan_busybox/tags/list | jq .
{% endhighlight %}

Something really interesting is that we can mount an external folder into the container so that we can modify the content or access it from the container:

{% highlight cs %}
docker run -v ${PWD}/my_volume/:/my_volume -ti busybox sh
{% endhighlight %}

Then inside the container we can see the files:

{% highlight cs %}
ls -ltr: show all the files
{% endhighlight %}

And even modify the external file since it is mounted inside:

{% highlight cs %}
echo "Inside" > /my_volume/Outside.txt
{% endhighlight %}

To only read it but not possible to modify it:

{% highlight cs %}
docker run -v ${PWD}/my_volume/:/my_volume:ro -ti busybox sh
{% endhighlight %}

We can also create a copy of certain files:

{% highlight cs %}
docker run -v ${PWD}/my_volume/Outside.txt:/my_volume/Inside.txt -ti busybox sh
{% endhighlight %}

### Networks

To find all the connections running in a container:

{% highlight cs %}
docker network ls
{% endhighlight %}

One of them is the bridge, the one we use to communicate containers together.

Create new network:

{% highlight cs %}
docker network create training-network
{% endhighlight %}

Create a web app and add it to the network:

{% highlight cs %}
docker run -d -P --name web --net training-network training/webapp python app.py
{% endhighlight %}

Create another app with the bash interactively and add it to the network:

{% highlight cs %}
docker run -ti --net training-network ubuntu bash
{% endhighlight %}

Now inside the second container, we install curl and dnsutils:

{% highlight cs %}
apt-get install -y curl
apt-get install -y dnsutils
{% endhighlight %}

With this we can do: ```dig web``` and we can query the other container from the second container: ```curl web:5000```

Inspect what containers any remote machine is running:

{% highlight cs %}
docker -H tcp://<machine-name>:<port-name> ps -a
{% endhighlight %}

### Managing containers

All this is nice but we have something called Docker compose that makes our life easier. If we create a **docker-compose.yml** file and put this:

{% highlight cs %}
 version: "2"

 services:
  web:
   image: training/webapp
   networks:
    - service_network
   command:
    - "python"
    - "app.py"
 
  web2:
   image: training/webapp
   networks:
    service_network:
     aliases:
      - web_no_two
   command:
    - "python"
    - "app.py"
 
  web3:
   image: training/webapp
   networks:
    - service_network
   command:
    - "python"
    - "app.py"
 
 networks:
  service_network:
{% endhighlight %}

We are telling docker compose to create 3 containers based on a given image and each will run the command. Also we are telling the network. For the second app we are also aliasing the network. Be careful with the tab position, it needs to match.

Now we can start all these containers at once:

{% highlight cs %}
docker-compose -f docker-compose.yml up
{% endhighlight %}

And stop them:

{% highlight cs %}
docker-compose -f docker-compose.yml down
{% endhighlight %}

We can tell compose to create services and run them in the background:

{% highlight cs %}
docker-compose -f docker-compose.yml up -d
{% endhighlight %}

Now let's connect inside one of the containers:

{% highlight cs %}
docker exec -ti mycompose_web_1 bash
{% endhighlight %}

We can again install curl and dnsutils. To do that, since they are new we have to do: ```apt-get update```. Now we are ready to connect from one container to the others.

This is awesome, we can tell one of the containers to scale to more containers:

{% highlight cs %}
docker-compose -f docker-compose.yml scale web2=2
{% endhighlight %}

## Deployment of an application using Docker and TeamCity

In order to deploy an application with Docker we need to push an image to a registry during the build. When we deploy the app, we will do docker run and pull the image from the registry.

These are the initial steps:

- First let's fork this repo into our own github account:  [https://github.com/codurance/simple_rest](https://github.com/codurance/simple_rest)
This is a simple web app written in Java. It uses something called Gradle that it is used to run Java apps.

- Now let's clone it into the machine with Docker:

{% highlight cs %}
git clone https://github.com/javflores/simple_rest
{% endhighlight %}

- Let's go to TeamCity now and set up a Project. [http://teamcity.training.codurance.io/project.html?projectId=Workstation8](http://teamcity.training.codurance.io/project.html?projectId=Workstation8)
We put a name of the project.
In VCS Roots we put the url of our github repo: [https://github.com/javflores/simple_rest.git](https://github.com/javflores/simple_rest.git)

### 1. Build 

Let's create a Build configuration that will be responsible to generate the executable.
In General Settings we specify the Artifact paths that this build needs:

{% highlight cs %}
docker/Dockerfile
build/distributions/simple_rest.tar
{% endlight %}

We set up a Build step of type Gradle, we tell the gradle file: in this case build.gradle.

Let's setup also a VCS trigger so that anytime there is a change in github master, this will trigger.

