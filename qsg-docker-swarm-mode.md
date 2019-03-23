# Quick Start Guide - Docker Swarm Mode
# Introduction

## _**Promises of Container and Swarm Mode**_

### **Promises of Container**
* Container enable easy deployment of our app, like a platform service, on anyone's hardware either ours or cloud providers', and it can be virtual or physical. 
* Container app deployed run the same whether they are Amazon, Azure, DigitalOcean, Linode, Racksapce, or Google Cloud.

### **Promises of Swarm Mode**
* Swarm provide platform features, to easily deploy and maintain dozens, or hundreds, or even thousands of containers across many servers or instances. 
* Swarm answer the following questions for team with only a couple of people or for you who are a solo developer:
  * How do we automate container lifecycle (deploy, start, restart, recreate, delete, update, and etc.)?
  * How can we easily scale out/in/up/down?
  * How can we ensure our containers are re-created if they fail?
  * How can we replace containers without downtime (blue/green deploy)?
  * How can we control/track where containers get started?
  * How can we create cross-node virtual networks?
  * How can we ensure only trusted servers run our containers?
  * How can we store secrets, keys, passwords and get them to the right container (and only that container)?

Note: Swarm Mode was released as version 1.12, it is not the prior "classic" Swarm which was an add-on component running inside of Docker that really just took the "docker run" commands and repeat out to multiple servers.

## _**Basic Concepts**_
How Swarm Mode works:  
* [How nodes work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/)
* [How services work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/)
* [Manage swarm security with PKI](https://docs.docker.com/engine/swarm/how-swarm-mode-works/pki/)
* [Swarm task states](https://docs.docker.com/engine/swarm/how-swarm-mode-works/swarm-task-states/)

Swarm is not enabled by default, such design goal is to ensure that none of the Swarm code would affect the exisitng daemon, tools and systems that already relying on Container, those existing would continue to function and not be interfered by Swarm now being part of the Docker. Once Swarm enabled, you could run the "docker swarm", "docker node", "docker service", "docker stack", and "docker secret" command.

## _**Built-In Orchestration**_
[DOCKER BUILT-IN ORCHESTRATION READY FOR PRODUCTION: DOCKER 1.12 GOES GA](https://blog.docker.com/2016/07/docker-built-in-orchestration-ready-for-production-docker-1-12-goes-ga/) blog explain the built-in orchestration architecture: 
* Swarm Mode Architectural Topology
* Manager to Manager Communication
* Manager to Worker Communication
* Worker to Worker Gossip

References:  
[Raft: Understandable Distributed Consensus](http://thesecretlivesofdata.com/raft/)  
[Docker 1.12 Swarm Mode Deep Dive Part 1: Topology](https://www.youtube.com/watch?v=dooPhkXT9yI)  
[Docker 1.12 Swarm Mode Deep Dive Part 2: Orchestration](https://www.youtube.com/watch?v=_F6PSP-qhdA)  
[Heart of the SwarmKit: Distributed Data Store, Topology Management & Object Model](https://www.youtube.com/watch?v=EmePhjGnCXY)  
[Slide: [Distributed Data Store](https://speakerdeck.com/aaronlehmann/heart-of-the-swarmkit-distributed-data-store)], [Slide: [Topology Management](https://speakerdeck.com/aluzzardi/heart-of-the-swarmkit-topology-management)], [Slide: [Object Model](https://speakerdeck.com/stevvooe/heart-of-the-swarmkit-object-model)]

# Initialize a Swarm
To verify whether Swarm is on, run the ```docker info``` command and look for the line "Swarm: \<status\>".
```
$ docker info
```

To initialize Swarm, run ```docker swarm init``` command, once intilialized, we will have a single node swarm with all the fetures and functionality out of the box.
```
$ docker swarm init
```
Under the hood this creates a Raft consensus group of one node. This first node has the role of manager, meaning it accepts commands and schedule tasks. As you join more nodes to the swarm, they will by default be workers, which simply execute containers dispatched by the manager. You can optionally add additional manager nodes. The manager nodes will be part of the Raft consensus group. We use an optimized Raft store in which reads are serviced directly from memory which makes scheduling performance fast.   
-- _by [DOCKER 1.12: NOW WITH BUILT-IN ORCHESTRATION!](https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/)_

![](https://docs.docker.com/engine/swarm/images/swarm-diagram.png)

When starting your first manager, Docker Engine will generate a new Certificate Authority (CA) and a set of initial certificates for you. After this initial step, every node joining the swarm will automatically be issued a new certificate with a randomly generated ID, and their current role in the swarm (manager or worker). These certificates will be used as their cryptographically secure node identity for the lifetime of their participation in this swarm, and will be used by the managers to ensure secure dissemination of tasks and other updates.  
-- _by [DOCKER 1.12: NOW WITH BUILT-IN ORCHESTRATION!](https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/)_

![](https://docs.docker.com/engine/swarm/images/tls.png)

To list nodes in the swarm, use ```docker node ls``` command. For the first time Swarm initialization, it will show one manager node is created, it is marked as leader, since we can only have one leader at a time amongst all managers.
```
$ docker node ls
ID                            HOSTNAME                STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
is6jj55v50fxog5a663u70lvx *   linuxkit-025000000001   Ready               Active              Leader              18.09.2
```

# Basic Command
## _**Docker Swarm Command**_
docker swarm commands are used for initialize, join or leave the swarm.
```
$ docker swarm --help

Usage:	docker swarm COMMAND

Manage Swarm

Commands:
  ca          Display and rotate the root CA
  init        Initialize a swarm
  join        Join a swarm as a node and/or manager
  join-token  Manage join tokens
  leave       Leave the swarm
  unlock      Unlock swarm
  unlock-key  Manage the unlock key
  update      Update the swarm

Run 'docker swarm COMMAND --help' for more information on a command.
```

## _**Docker Node Command**_
docker node commands are used for bringing your servers in and out of the swarm, promoting node from workers to managers, or demoting them from managers back down to workers.

```
$ docker node --help

Usage:	docker node COMMAND

Manage Swarm nodes

Commands:
  demote      Demote one or more nodes from manager in the swarm
  inspect     Display detailed information on one or more nodes
  ls          List nodes in the swarm
  promote     Promote one or more nodes to manager in the swarm
  ps          List tasks running on one or more nodes, defaults to current node
  rm          Remove one or more nodes from the swarm
  update      Update a node

Run 'docker node COMMAND --help' for more information on a command.
```

## _**Docker Service Command**_
docker service commands are used in swarm instead of docker container commands, this was centered arround the idea that we didn't want to break existing docker container commands functionality. 

```
docker service --help

Usage:	docker service COMMAND

Manage services

Commands:
  create      Create a new service
  inspect     Display detailed information on one or more services
  logs        Fetch the logs of a service or task
  ls          List services
  ps          List the tasks of one or more services
  rm          Remove one or more services
  rollback    Revert changes to a service's configuration
  scale       Scale one or multiple replicated services
  update      Update a service

Run 'docker service COMMAND --help' for more information on a command.
```

The docker container commands is built from ground up as a single host solution focus on local containers. Whereas, when we start talking about a cluster, we really just provide the requirements at the swarm in the form of services, where swarm will orchestrate how it needs to lay all that out, on which nodes they need to be, and we just know that it got our back.
* we don't focus on individual nodes
* we don't actually name each of them.
* we don't really individually go to each node and start up individual container.

# Create and Scale the Service Locally

## _**Create a Service**_
Use ```docker service create``` command to start the Alpine image and have it running ping to hit 8.8.8.8, give it something to do while we investigate what is happening in the background. The output shows the service ID (i.e. 1htr8h09dpcru5um0x9u2iewn) of the created service.
```
$ docker service create alpine ping 8.8.8.8
yvwy1t4onjk7ibev27slx8nl2
```
## _**Investigate the "docker service create" Command**_

### **List all the Services**
To see the list of services in the node ??? is it in the node or in the swarm ???, use ```docker service ls``` command. (Note: You might see other services which you have created out of this tutorial)

```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
yvwy1t4onjk7        quirky_darwin       replicated          1/1                 alpine:latest
```

As for this tutorial, the output shows one service listed, which was [created previously](##-Create-a-Service),  given a random name (i.e. quirky_darwin) since we didn't specified, and have 1/1 replica spun up.  

The goal of the orchestrator is to make the replica #/# numbers match.
* The number on the left is how many are actually running. 
* The number on the right is how many you have specified for it to run.

### **List all the Tasks of the Service**
To see the list of tasks of a service, use ```docker service ps``` command.

```
$ docker service ps quirky_darwin
ID                  NAME                IMAGE               NODE                    DESIRED STATE       CURRENT STATE            ERROR               PORTS
q2lve12rj1n5        quirky_darwin.1     alpine:latest       linuxkit-025000000001   Running             Running 53 seconds ago
```

As for this tutorial, the output shows one task listed, which was [created previously](##-Create-a-Service), given a name of an increment on the service name, and with the node component indicating which server the task is runnning on.

### **List all the Containers**
To see the list of containers, use ```docker container ls``` command.

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
01f7ac967283        alpine:latest       "ping 8.8.8.8"      2 minutes ago       Up 2 minutes                            quirky_darwin.1.q2lve12rj1n5x2oubuhoq5rgn
```

As for this tutorial, the output shows one container listed, which was [created previously](##-Create-a-Service), notice that the orchestration of Swarm has added some information to the container name. 

## _**Scale Up a Service**_

Use ```docker service update``` command to scale up the service by changing the number of replicas.
```
$ docker service update quirky_darwin --replicas 3
```

## _**Investigate the "docker service update" Command**_

### **List all the Services**
To see the update on the service, use ```docker service ls``` command. 

```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
yvwy1t4onjk7        quirky_darwin       replicated          3/3                 alpine:latest
```

As for this tutorial, the output shows 3/3 replicas spun up, 1 was [created previously](##-Create-a-Service), and the other 2 were [created recently from the update](##-Scale-Up-a-Service). You might need to run the command a few times if the task is taking longer time to start up, the output might shows 0/3, 1/3, 2/3, 3/3 replica incrementally over the period of starting up.

### **List all the Tasks of the Service**
To see the update on the tasks, use ```docker service ps``` command. 

```
$ docker service ps quirky_darwin
ID                  NAME                IMAGE               NODE                    DESIRED STATE       CURRENT STATE            ERROR               PORTS
q2lve12rj1n5        quirky_darwin.1     alpine:latest       linuxkit-025000000001   Running             Running 3 minutes ago
4f5s9qrlclvu        quirky_darwin.2     alpine:latest       linuxkit-025000000001   Running             Running 52 seconds ago
35aysj4gk09l        quirky_darwin.3     alpine:latest       linuxkit-025000000001   Running             Running 52 seconds ago
```

As for this tutorial, the output shows 3 tasks is running, where 2 were [created recently from the update](##-Scale-Up-a-Service).

## _**Ensure Consistent Availability**_

This section is to demonstrate the reaction of Swarm when one of the running container is down. As to simulate this scenario, we forcefully took away the running container behind the back of Swarm.

Notice that now we have 3 containers after updating the service to have 3 replicas
```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
445b7cd47520        alpine:latest       "ping 8.8.8.8"      About a minute ago   Up About a minute                       quirky_darwin.3.35aysj4gk09lqo5ib49sb7xpc
66c369c120bf        alpine:latest       "ping 8.8.8.8"      About a minute ago   Up About a minute                       quirky_darwin.2.4f5s9qrlclvug8l31dlt3ge5l
01f7ac967283        alpine:latest       "ping 8.8.8.8"      4 minutes ago        Up 4 minutes                            quirky_darwin.1.q2lve12rj1n5x2oubuhoq5rgn
```

(Note: the next two commands need to be run immediately one after another to see the effect.)

To simulate the scenario, use ```docker container rm``` to forcefully remove a running container behind the back of Swarm.
```
$ docker container rm -f quirky_darwin.1.lxm7kl47eqixai3czs6233m4d
```

To see the effect on the service, use ```docker service ls``` command. 
The output shows 2/3 replicas spun up .
```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
yvwy1t4onjk7        quirky_darwin       replicated          2/3                 alpine:latest
```

Wait for a short while, Swarm is going to identify the removed container, it is going to launch a new one within seconds to replace the one that went down.

This is the responsibilities of the orchestration system, to make sure that the services you specified are always running, and if they fail, it recovers from that failure. 

To see the effect on the service, use ```docker service ls``` command. 
The output shows 3/3 replicas spun up .
```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
yvwy1t4onjk7        quirky_darwin       replicated          3/3                 alpine:latest
```

To see the effect on the tasks, use ```docker service ps``` command. 
The output shows the history of the first task, it had one that failed, and it started a new one few seconds ago.
```
$ docker service ps quirky_darwin
ID                  NAME                  IMAGE               NODE                    DESIRED STATE       CURRENT STATE            ERROR                         PORTS
t5t0ncm7fpdf        quirky_darwin.1       alpine:latest       linuxkit-025000000001   Running             Running 49 seconds ago
q2lve12rj1n5         \_ quirky_darwin.1   alpine:latest       linuxkit-025000000001   Shutdown            Failed 54 seconds ago    "task: non-zero exit (137)"
4f5s9qrlclvu        quirky_darwin.2       alpine:latest       linuxkit-025000000001   Running             Running 2 minutes ago
35aysj4gk09l        quirky_darwin.3       alpine:latest       linuxkit-025000000001   Running             Running 2 minutes ago
```

Whenever we use any of these docker service commands, we are actually telling an orchestration system to put those jobs specified by docker service commands into the queue, and perform those jobs when the orchestration system can get to it.

To remove all those corresponding containers of the service on Swarm, you have to remove the service, use ```docker service rm``` command.
```
$ docker service rm quirky_darwin
```

If you run ```docker service ls``` and ```docker container ls```, you will see that the service and its containers were removed.


## _**Difference between "docker container update" and "docker service update"**_

Before we discover the differences, we clearly say that container restart is unavoidable for certain changes to be updated, and definitely there are also some changes can be updated without the need to take down the container.

### **"docker container update" command**

Container created and started by ```docker container run``` command is being used on a single local machine, test server or dev server. Under such circumstances, a single container is running without the concerns to ensure its availability. Taking down the only single running container is expected to update those changes which requires a restart.

Therefore, the ```docker container update`` command is designed to only cater for changes that does not require restart, most of the options are related to limiting and controlling resource usage on the container such as RAM and CPU. Wheraeas, those changes that require restart will go through the manual way of taking down the container for the changes to be applied. 

```
$ docker container update --help

Usage:	docker container update [OPTIONS] CONTAINER [CONTAINER...]

Update configuration of one or more containers

Options:
...
(please run the command to see the Options available)
```

### **"docker service update" command**

Service created and started by ```docker service create``` command is being used in production server. Under such circumstances, more than one replicated containers of the service are running with the concerns to ensure its availability. Swarm which managing the service is able to replace those containers with updated changes without taking the entire service down. Swarm could technically take down one at a time to make the changes and do a rolling update, the blue green pattern. 

Therefore, the ```docker service update``` command allow a lot more options including those changes that require restart, where Swarm will intelligently make sure that the updating process is in a pattern that ensures consistent availability.

```
$ docker service update --help

Usage:	docker service update [OPTIONS] SERVICE

Update a service

Options:
... 
(please run the command to see the Options available)
```

Reference:  
[Deploy services to a swarm](https://docs.docker.com/engine/swarm/services/)