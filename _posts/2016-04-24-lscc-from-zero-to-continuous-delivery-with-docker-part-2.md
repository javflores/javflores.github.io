---
layout: post
title:  From zero to Continuous Delivery with Docker, day 2
---

Today was a really useful day but really hard for me. A lot more things came into play to create a distributed system with zero downtime.
I'll try my best to keep the notes I've taken. This was the programme:

- Implement a continuous deployment with Docker Swarm
- Explore techniques and strategies for achieving zero downtime
- Blue/Green Deployment
- Canary Release

## Distributed system

If we need to scale a system (for example with high demand), how do we manage ips, restarting and rest of clumsy stuff?
We want to avoid the manual setup, everything should be automated.

To solve this challenge we need two ingredients:

- Docker swarm. It is a native clustering for Docker, it turns a pool of Docker hosts into a single virtual Docker host. 
Docker swarm cluster is composed of master and agents. A Docker swarm is aware of every machine running docker in the cluster. How?

- Consul. It is a service discovery system. It keeps information about where an agent is, and it tells to register in a cluster. 
Agents put themselves in the database key value. They do that by telling the Consul cluster, Consul is basically a distributed key value store, 
a cluster of agents and managers responsible to keep up to date the info of IP addresses and port numbers of swarm agents.
Consul uses a key value store, machine, IP address and port number.

When deploying the application, Swarm manager asks to consul which are the available agents.

In the practice, Robert created 3 consul servers and configure the cluster with them. There is a UI where we can check:
[http://consul-0.training.codurance.io:8500/ui/#/training/services/consul](http://consul-0.training.codurance.io:8500/ui/#/training/services/consul)

Then we added our machine as a consul agent:
In our machine we already had a file with the consul configuration:

{% highlight cs %}
nano /var/data/consul_config/consul_agent.json
{% endhighlight %}

The content of the file:

{% highlight cs %}
{
  "datacenter": "training",
  "node_name": "Workstation 8",
  "advertise_addr": "10.2.0.110",
  "retry_join": ["10.2.0.191","10.2.0.126","10.2.0.41"],
  "data_dir": "/data",
  "log_level": "INFO",
  "client_addr": "0.0.0.0",
  "leave_on_terminate": true
}
{% endhighlight %}

In this file, we tail that we are joining the same datacenter. We have our local IP addres (advertise_addr).
Retry join tells what it is the consul servers cluster, when announcing it exists. Finally we say leave the cluster if the process terminates.

Let's create now the consul agent:

{% highlight cs %}
docker run -d -v /var/data/consul_config/:/config -v /var/data/consul_data:/data 
  --net=host --label consul_agent --name consul_agent gliderlabs/consul-agent:0.6
{% endhighlight %}

Now in the UI we can check that our agent is being added, [http://consul-0.training.codurance.io:8500/ui/#/training/nodes/Workstation 8](http://consul-0.training.codurance.io:8500/ui/#/training/nodes/Workstation 8).

To stop the agent:

{% highlight cs %}
docker stop 073
{% endhighlight %}

073 is part of the image id.

Now that we have a consul agent in our machine, let's configure the swarm cluster. 
Robert configured a swarm manager, so then we just add a new swarm agent in our machine by doing:

{% highlight cs %}
docker run -d  --label swarm_agent --name swarm_agent swarm:1.0.1 join --advertise=10.2.0.110:2375 consul://10.2.0.110:8500/training_swarm
{% endhighlight %}

The ip address is the one in our local machine.

### Plugin Swarm to TeamCity

Yesterday we deployed directly to our machine. Today we deployed from TeamCity into our just configured Docker swarm.
To do that just modify the Deploy build step to run this command:

{% highlight cs %}
#!/bin/bash
set -e
export IMAGE_VERSION=$(cat release.version)
export DOCKER_HOST=tcp://swarm-manager.training.local:4567
docker-compose scale workstation-8=0
docker-compose scale workstation-8=1
{% endhighlight %}

The Host is now the swarm manager. We use docker-compose to put the current container (=0) and then run again the new containers (=1).

## Service Discovery

Now we have a distributed system of machines running containers with a given random port and IP address, so that we need some service discovery to know 
where we have to direct a request.

We use Consul facility for service discovery. We need to register a name for our application as a service. To do that we use another container running something called Registrator.
[http://gliderlabs.com/registrator/latest/user/services/](http://gliderlabs.com/registrator/latest/user/services/) 
 
Now let's create the registrator:

{% highlight cs %}
docker run -d --net=host -v /var/run/docker.sock:/tmp/docker.sock 
  --label registrator --name registrator gliderlabs/registrator:v7 -ip 10.2.0.110 consul://10.2.0.110:8500
{% endhighlight %}

In order to listen to events the container needs to publish some events thats why we have the .scock file above. 
To plug the registrator to the container we need to include the ip addresses.

Then in the docker-compose.yml we changed the compose command to have:

{% highlight cs %}
version: "2"

services:
  workstation-8:
    image: registry.training.local/workstation-8:${IMAGE_VERSION} 
    network_mode: bridge
    environment:
      - "SERVICE_NAME=juan"
    ports:
      - "4567"
{% endhighlight %}

This means we have created in addition to workstation-8, another service called Juan which will be used by Consul.

### Proxying, Nginx

Now we have a bunch of container running different services, one of them is a registrator connected to the docker swarm agent.
We need to direct external traffic to it and for that we need a proxy. We will use Nginx load balancer to do that.

For that in the workshop we forked [https://github.com/codurance/nginx](https://github.com/codurance/nginx).

In the NGinx config we have:

{% highlight cs %}
upstream dummy {
        least_conn;
        server 127.0.0.1:65535; # force a 502
    }

    server {
        listen 80 default_server;
        server_name _;

        location /dummy/ {
            proxy_pass http://dummy/;
        }
   }
 {% endhighlight %}
 
 Basically a request to /dummy/ is directed to the upstream. The upstream defines a list of ip addresses.
 
 We have also a Dockerfile that creates the nginx application.
 
Every consul agent will communicate with Nginx by means of a consul template. If there is a problem with the container, nginx will manage it.
We have a consul template written in Go which defines how to access the containers:

{% highlight cs %}
worker_processes 1;
events {
    worker_connections 1024;
}

daemon off;
pid /var/run/nginx.pid;

http {
    include mime.types;
    default_type application/octet-stream;

    gzip on;
    sendfile on;
    keepalive_timeout 60;


    {{range services}}
    upstream {{.Name}} {
        least_conn;
        {{range service .Name}}server {{.Address}}:{{.Port}} max_fails=3 fail_timeout=60 weight=1;
        {{else}}server 127.0.0.1:65535; # force a 502{{end}}
    }
    {{else}}
    upstream dummy {
        least_conn;
        server 127.0.0.1:65535; # force a 502
    }
    {{end}}


    server {
        listen 80 default_server;
        server_name _;

        {{range services}}
        location /{{.Name}}/ {
             proxy_pass http://{{.Name}}/;
        }
        {{else}}
        location /dummy/ {
            proxy_pass http://dummy/;
        }
        {{end}}
    }
}
{% endhighlight %}
 
Now we need to tell TeamCity to build all these. 

- We need to create another configuration (Build Nginx) to trigger this thing. In General settings we put artifact paths: release.version
In VCS Setting the forked repo, an a trigger to Github. The build step:

{% highlight cs %}
#!/bin/bash
set -e
docker build --tag registry.local.training/juan-nginx:%build.number% .
docker push registry.local.training/juan-nginx:%build.number%
echo %build.number% > release.version
{% endhighlight %}

- We need another build configuration with a step to run the container (Deploy Nginx):

{% highlight cs %}
#!/bin/bash
set -e
export VERSION=$(cat release.version)
export DOCKER_HOST=tcp://workstation-8.training.local:2375
docker rm -f nginx_api || echo "warning NGINX is not running yet"
docker run -d -p 80:80 --add-host=consul:172.17.42.1  -e SERVICE_IGNORE=true--name nginx_api registry.training.local/api/juan-nginx:${VERSION}
{% endhighlight %}

A trigger with the previous step and finally a couple of dependencies.

Finally we can go to the browser and do this request to see the result:
[http://workstation-8.training.codurance.io/juan/hello](http://workstation-8.training.codurance.io/juan/hello)

## Zero Downtime

When we are deploying some new version of the code we may introduce some latency where the app won't be available.
There are two strategies to solve this situation.

### Blue Green Deployment, [http://martinfowler.com/bliki/BlueGreenDeployment.html](http://martinfowler.com/bliki/BlueGreenDeployment.html)

To have an active and inactive we need to change the Docker compose to this:

{% highlight cs %}
version: "2"

services:
  workstation-0-blue:
    image: "registry.training.local/workstation-0:${IMAGE_VERSION}"
    network_mode: bridge
    environment:
      - "SERVICE_NAME=workstation-0"
      - "SERVICE_TAGS=blue"
    ports:
      - "4567"
  workstation-0-green:
    image: "registry.training.local/workstation-0:${IMAGE_VERSION}"
    network_mode: bridge
    environment:
      - "SERVICE_NAME=workstation-0"
      - "SERVICE_TAGS=green"
    ports:
      - "4567"
{% endhighlight %}

We have defined two services, blue and green. We provide an image version to the Docker image of each and also a service tag for the environment.

In the Consule template we read the active version:

{% highlight cs %}
worker_processes 1;
events {
    worker_connections 1024;
}

daemon off;
pid /var/run/nginx.pid;

http {
    include mime.types;
    default_type application/octet-stream;

    gzip on;
    sendfile on;
    keepalive_timeout 60;

    {{range services}}
    upstream {{.Name}} {
        least_conn;
        {{$activeZone := key (printf "%s-active-zone" .Name)}}
        {{if $activeZone}}
            # active zone {{$activeZone}}
            {{range service (printf "%s.%s" $activeZone .Name)}}server {{.Address}}:{{.Port}} max_fails=3 fail_timeout=60 weight=1;
            {{else}}server 127.0.0.1:65535; # force a 502{{end}}
        {{else}}
            {{range service .Name}}server {{.Address}}:{{.Port}} max_fails=3 fail_timeout=60 weight=1;
            {{else}}server 127.0.0.1:65535; # force a 502{{end}}
        {{end}}
    }
    {{else}}
    upstream dummy {
        least_conn;
        server 127.0.0.1:65535; # force a 502
    }
    {{end}}


    server {
        listen 80 default_server;
        server_name _;

        {{range services}}
        location /{{.Name}}/ {
             proxy_pass http://{{.Name}}/;
        }
        {{else}}
        location /dummy/ {
            proxy_pass http://dummy/;
        }
        {{end}}
    }
}
{% endhighlight %}

Finally in the deployment step in TeamCity we run the following commands:

{% highlight cs %}
#!/bin/bash
set -e

export ACTIVE_ZONE=$(curl -sf http://consul-0.training.local:8500/v1/kv/workstation-0-active-zone?raw)
if [ "${ACTIVE_ZONE}" = "blue" ]; then 
   export INACTIVE_ZONE="green"
else 
   export INACTIVE_ZONE="blue"
fi

echo "Active Zone   : ${ACTIVE_ZONE}"
echo "Inactive Zone : ${INACTIVE_ZONE}"

export IMAGE_VERSION=$(cat release.version)
export DOCKER_HOST=tcp://swarm-manager.training.local:4567
    
docker-compose scale workstation-0-${INACTIVE_ZONE}=4 
echo "Waiting for new zone (${INACTIVE_ZONE})"
sleep 10

echo "Switch to new zone"
curl -XPUT -d "${INACTIVE_ZONE}" -sf http://consul-0.training.local:8500/v1/kv/workstation-0-active-zone

echo "Remove previous zone (${ACTIVE_ZONE})"
docker-compose scale workstation-0-${ACTIVE_ZONE}=0
{% endhighlight %}

### Canary Release, [http://martinfowler.com/bliki/CanaryRelease.html](http://martinfowler.com/bliki/CanaryRelease.html)

In this case we modify the Docker compose file in our application:

{% highlight cs %}
version: "2"

services:
  workstation-8-blue:
    image: registry.training.local/workstation-8:${BLUE_IMAGE_VERSION} 
    network_mode: bridge
    environment:
      - "SERVICE_NAME=juan"
      - "SERVICE_TAGS=blue_v${BLUE_IMAGE_VERSION}"
    ports:
      - "4567"
      
  workstation-8-green:
    image: registry.training.local/workstation-8:${GREEN_IMAGE_VERSION} 
    network_mode: bridge
    environment:
      - "SERVICE_NAME=juan"
      - "SERVICE_TAGS=green_v${GREEN_IMAGE_VERSION}"
    ports:
      - "4567"
{% endhighlight %}

In this case we don't need to modify the consule template.
We finally just need to modify the Deployment build command with:

{% highlight cs %}
#!/bin/bash
set -e

export NEW_IMAGE_VERSION=$(cat release.version)
export OLD_IMAGE_VERSION=$(curl -sf http://consul-0.training.local:8500/v1/kv/workstation-0-active-version?raw)

export OLD_VERSION_ZONE=$(curl -sf http://consul-0.training.local:8500/v1/kv/workstation-0-active-zone?raw)
if [ "${OLD_VERSION_ZONE}" = "blue" ]; then 
   export NEW_VERSION_ZONE="green"
   export BLUE_IMAGE_VERSION="${OLD_IMAGE_VERSION}"
   export GREEN_IMAGE_VERSION="${NEW_IMAGE_VERSION}"
else 
   export NEW_VERSION_ZONE="blue"
   export BLUE_IMAGE_VERSION="${NEW_IMAGE_VERSION}"
   export GREEN_IMAGE_VERSION="${OLD_IMAGE_VERSION}"
fi

echo "Old Version Zone: ${OLD_VERSION_ZONE} (version $OLD_IMAGE_VERSION)"
echo "New Version Zone: ${NEW_VERSION_ZONE} (version $NEW_IMAGE_VERSION)"

export DOCKER_HOST=tcp://swarm-manager.training.local:4567
    
docker-compose scale workstation-0-${OLD_VERSION_ZONE}=%old_version_size% 
docker-compose scale workstation-0-${NEW_VERSION_ZONE}=%new_version_size%

if [ true = %switch_to_new_version% ]; then
    curl -XPUT -d "${NEW_IMAGE_VERSION}" -sf http://consul-0.training.local:8500/v1/kv/workstation-0-active-version
    curl -XPUT -d "${NEW_VERSION_ZONE}" -sf http://consul-0.training.local:8500/v1/kv/workstation-0-active-zone
fi
{% endhighlight %}

That was it for the course. Very comprenhensive and practical workshop on how to use Docker as a tool to achieve Continuous Deployment and enjoy the Ops culture being a Developer.
Thanks a lot to Robert and Codurance to do this.

