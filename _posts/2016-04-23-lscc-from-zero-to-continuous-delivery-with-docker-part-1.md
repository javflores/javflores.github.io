---
layout: post
title:  LSCC Workshop: From zero to Continuous Delivery with Docker (Part 1)
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
``` docker version ```

More info about the Docker daemon:
``` docker info ```

Help is really useful:
``` docker help``` Example: docker help run

Find what images we've got downloaded
``` docker images ```

Images are stored in registries, the registry by default is DockerHub. To get an image into our machine:
``` docker pull ubuntu ```

That is the same as ``` docker pull library/ubuntu```

In general the convention in the registry is ```[username/]repository:tag```

We can navigate to Docker hub and search for Linux Centos: https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=centos&starCount=0
And pull one of them.

If we want to have several images of the same thing like ubuntu you can put a tags. For example:
```docker pull library/ubuntu:14.04```
Suggestion, always use tags.

We can remove images (It is a dangerous operation)
```docker rmi ubuntu:14.04```
And with this:
```docker rm $(docker ps -aq)```

To run a command:
```docker run busybox sh -c "echo Hello World"```
In this case it pulled an image from DockerHub called busybox and run inside the container a shell, Hello world.

We can run a command based on name or label:
```docker run --name juan busybox sh -c "echo Hello World"```
``` docker run --label juan busybox sh -c "echo Hello World"```

Some commands require to use the imageId, but we don't need to provide the full imageId, but the first two numbers or 3 numbers.

To get info about an image:
```docker inspect <image-id>```
And to see the logs:
```docker logs <image-id>```

Actually with the docker inspect command we can find where the log files of the process are and then we can see them with: 
```sudo cat <file-path>```.

Let's run a more complicated program that prints Hello world in a while loop:
```docker run -d --name loop_ubuntu ubuntu /bin/sh -c  "while true; do echo hello world; sleep 1; done"```
We can kill it:
```docker kill <id>```

Or more gracefully:
```docker stop <id>```

Let actually run an application, we can use one of the training apps provided by Docker, in this case Python:
```docker run -d --name web training/webapp python app.py```

With -P it will expose a port number externally:
```docker run -d -P training/webapp python app.py```

To map a external port to an internal one:
```docker run -d -p 8080:5000 -p 8081:5001 training/webapp python app.py```

Let's see what containers we've got running:
```docker ps -a -q```

We can use this command to delete the running containers:
```docker ps -a -q | xargs docker rm -f```

To run a container interactively:
```docker run -ti```

For example: 
```docker run -ti busybox sh```
Now you are in the container and you can run the terminal. You can ```echo "Hello world" > HelloWorld.txt```
We can also:
```
mkdir my_busybox
cd my_busybox
nano Dockerfile
```

This way we have created a Dockerfile from inside the container. The Docker file can execute linux commands for us and much more.

Given a folder with a DockerFile we can build a container:
```docker build .``` 
It is untagged so we can:
```docker tag <tag>```

The following sets a new image based on that container or add changes to an image, container.
```docker commit -m "My Busy Box" -a "Juan Vicaria" 42 my_busybox:v5``` (we found 42 the image id of one container with docker ps -a)

Then we can run the container, even providing a parameter into the Docker run:
docker run my_bussybox:v5 -c "echo Nothing"

### Registries

We can create private registries to hold images. In the workshop we had a private one ```registry.training.local\```

Before we tagged the image:
```docker tag my_bussybox:v5 registry.training.local/juan_busybox```

Then pushing the image given that my machine is connected to the private repo with the VPN:
```docker push registry.training.local/juan_busybox```

Finding all the images in a private repository:
```curl registry.training.local/v2/_catalog | jq .```

Finding all the history of an image and all the tags:
```curl registry.training.local/v2/juan_busybox/tags/list | jq .```

Something really interesting is that we can mount an external folder into the container so that we can modify the content or access it from the container:
```docker run -v ${PWD}/my_volume/:/my_volume -ti busybox sh```
Then inside the container we can see the files:
```ls -ltr: show all the files```
And even modify the external file since it is mounted inside:
```echo "Inside" > /my_volume/Outside.txt```

To only read it but not possible to modify it:
```docker run -v ${PWD}/my_volume/:/my_volume:ro -ti busybox sh```

We can also create a copy of certain files:
```docker run -v ${PWD}/my_volume/Outside.txt:/my_volume/Inside.txt -ti busybox sh```

### Networks

To find all the connections running in a container:
```docker network ls```
One of them is the bridge, the one we use to communicate containers together.

Create new network:
 ```docker network create training-network```

Create a web app and add it to the network:
```docker run -d -P --name web --net training-network training/webapp python app.py```

Create another app with the bash interactively and add it to the network:
```docker run -ti --net training-network ubuntu bash```

Now inside the second container, we install curl and dnsutils:
```apt-get install -y curl```
```apt-get install -y dnsutils```

With this we can do: ```dig web``` and we can query the other container from the second container:
```curl web:5000```

Inspect what containers any remote machine is running:
docker -H tcp://<machine-name>:<port-name> ps -a

### Managing containers

All this is nice but we have something called Docker compose that makes our life easier. If we create a **docker-compose.yml** file and put this:
```
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
```

We are telling docker compose to create 3 containers based on a given image and each will run the command. Also we are telling the network.
For the second app we are also aliasing the network. Be careful with the tab position, it needs to match.

Now we can start all these containers at once:
```docker-compose -f docker-compose.yml up```
And stop them:
```docker-compose -f docker-compose.yml down```

We can tell compose to create services and run them in the background:
```docker-compose -f docker-compose.yml up -d```

Now let's connect inside one of the containers:
```docker exec -ti mycompose_web_1 bash```
We can again install curl and dnsutils. To do that, since they are new we have to do:
```apt-get update```
Now we are ready to connect from one container to the others.

This is awesome, we can tell one of the containers to scale to more containers:
```docker-compose -f docker-compose.yml scale web2=2```

## Deployment of an application using Docker and TeamCity

In order to deploy an application with Docker we need to push an image to a registry during the build. When we deploy the app, we will do docker run and pull the image from the registry.

These are the steps:

- First let's fork this repo into our own github account: https://github.com/codurance/simple_rest
This is a simple web app written in Java. It uses something called Gradle that it is used to run Java apps.

- Now let's clone it into the machine with Docker:
```git clone https://github.com/javflores/simple_rest```

- Let's go to TeamCity now and set up a Project. http://teamcity.training.codurance.io/project.html?projectId=Workstation8
We put a name of the project.
In VCS Roots we put the url of our github repo: https://github.com/javflores/simple_rest.git

- **1. Build**: Let's create a Build configuration that will be responsible to generate the executable.
In General Settings we specify the Artifact paths that this build needs:
```
docker/Dockerfile
build/distributions/simple_rest.tar
```
We set up a Build step of type Gradle, we tell the gradle file: in this case build.gradle.

Let's setup also a VCS trigger so that anytime there is a change in github master, this will trigger.

- **2. Release**: This will publish a docker container in our machine.
To execute this we need a trigger which is the previous step, 1. Build.

The Build Steps we need here are two:
 - A command line step to build a docker container and push it to the registry, the tag is based on the build number:
 ```
 #!/bin/bash
 set e
 docker build --tag registry.training.local/workstation-8:%build.number% .
 docker push registry.training.local/workstation-8:%build.number%
 ```
 
 -A command line to publish the release version:
 ```
 echo %build.number% > release.version
 ```
We need to set a snapshot dependency on the previous step and it needs the artifacts dependencies from the previous build (mark Clean destination paths before downloading artifacts, so we start clean): 
```
Dockerfile
simple_rest.tar
```
In General setting we tell the artifacts paths that this step will publish:
```
release.version
```

- **3. Deploy**: This will publish a docker container in our machine.
To execute this we need a trigger which is the previous step, 2. Release.
Here only one Build step which again is a command line with the following:
```
#!/bin/bash
set -e
export IMAGE_VERSION=$(cat release.version)
export DOCKER_HOST=tcp://workstation-8.training.local:2375
docker rm -f workstation-8 || echo "writing application is out running"
docker run -d -p 80:4567 --name workstation-2 registry.training.local/workstation-8:${IMAGE_VERSION}
```

First we check for errors, we create a couple of variables. Then we delete the previous container that existed in our machine.
The first time it won't exit so we just do echo. Finally we simply run the container.

Now we can do any changes in github or hit Run in the first step in teamcity. We should be able to run in our browser: localhost/hello and localhost/healthcheck
Also in our machine we can check that the container was created and if it running:
```
docker images
docker ps
```

We can also do ```curl localhost/hello``` and we should see *Hello world*.

The last thing of the day was to use Docker compose with the team city setup.
Everything is the same but:

- In step 1, we publish another artifact, the docker-compose.yml:
```
docker/Dockerfile
docker/docker-compose.yml
build/distributions/simple_rest.tar
```

- In step 2 we publish that artifact again:
```
release.version
docker-compose.yml
```

- In step 3, in the Build step add the following:
```
#!/bin/bash
set -e
export IMAGE_VERSION=$(cat release.version)
export DOCKER_HOST=tcp://workstation-8.training.local:2375

docker-compose down
docker-compose up -d
```
Really simple, just stop containers define in the  Docker compose file and bring them up again.


That was it for the day. Quite a day, too much to process.
Much more tomorrow. 
Thanks to Robert and Codurance to run this great workshop. I'm enjoying and finally get on better with Docker.

