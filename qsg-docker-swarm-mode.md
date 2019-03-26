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
To see the list of services in the swarm, use ```docker service ls``` command.

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

# Create 3-Node Swarm Cluster

We can’t do this with the built-in Docker on local machine because it is only a single node (host, instance, or OS), we need three node.

Options:
1. play-with-docker.com
   * only needs a browser to get started
   * comes with docker preinstalled
   * only takes seconds to provision multiple machines
   * require no investment at your end
   * every four hours will reset and wipe all the work you have done
2. docker-machine on local + VirtualBox 
   * require a local machine with 8GB of RAM
   * use docker-machine which by default works with VirtualBox locally
   * each virtual machine probably needs 1GB of RAM
3. DigitalOcean cloud + Docker Installed
   * cheapest and easiest service to get started
   * nice and fast running everything on SSD
   * very similar to production setup
4. Other cloud + Docker Installed
   * there are many ways to get Docker installed
     * you can use docker-machine, which has built-in drivers that allow you to provision Amazon instances, or Azure instances, DigitalOcean Droplets, or Google Compute nodes and etc
     * you can install Docker with an automated script from get.docker.com on any place you can get a Linux virtual machine

Note:  
* You can install docker on multiple platform following [the best installation path for each platform](https://docs.docker.com/install/).  
* docker-machine is an automation tool for provisioning virtual machines with Docker installed and setup both locally and on Internet. It is installed along with Docker for Windows and Docker for Mac, it need to be downloaded and installed manually for Linux.
  * you may use docker-machine to save you a few steps on provisioning and installing, especially for cloud service like Amazon and Azure which required you to provision a lot of different things before you can get to the provisioning of virtual machine.
  * you probably not going to use docker-machine in production, it is a tool to simply automate dev and test environments, it was never really designed to setup all of the production settings you might need for a multi-node swarm.

## _**Step 1 ~ Prepare 3 Instances with Docker Installed**_

### **Option 1: play-with-docker.com**

1. Getting to the point where Docker installed
    * Login into play-with-docker.com
    * To setup three nodes, click the "ADD NEW INSTANCE" button three times to create three nodes, i.e node1, node2, and node3. 
    * Nodes created automatically with Docker installed, and with specific ports of Swarm opened.

2. Make sure the node is working
    * To access node1, select node1 from the list on the left
    * To test node1, run the command inside the console-like section
      ```
      $ docker info
      ... 
      (output show the docker information)
      
      $ ping node2
      ... 
      (output show successful ping log)
      ```
      similarly, access and test both node2 and node3

### **Option 2: docker-machine on local + VirtualBox**

1. Getting to the point where Docker installed
    * Follow the [official installation guide](https://docs.docker.com/machine/install-machine/) to install docker-machine. The version number shown on the guide might not be the latest, please visit [docker machine github release page](https://github.com/docker/machine/releases), obtain the latest release version number to replace the version number provided by the official guide.
    * Download and install VirtualBox from https://www.virtualbox.org.
    * To setup three nodes, run  
      ```
      $ docker-machine create node1
      $ docker-machine create node2
      $ docker-machine create node3
      ```
    * Nodes created automatically with Docker installed, and with specific ports of Swarm opened.

2. Make sure the node is working
    * To access node1, run  
      ```
      $ docker-machine ssh node1
      ... 
      (remotely log in into node1 with remote terminal access)
      ```
      OR
      ```
      $ eval $(docker-machine env node1)
      ... 
      (set environment variables for the local terminal to talk to node1)
      ```
    * To test node1, run
      ```
      $ docker info
      ... 
      (output show the docker information)
      
      $ ping node2
      ... 
      (output show successful ping log)
      ```
      similarly, access and test both node2 and node3
    
    Note:
    ```docker-machine env node1``` command will output few environment variables export commands which are used for enabling the communication between local terminal and node1, you may copy and paste those command to .bash_profile or .bashrc if you need it to be executed when logged in to the shell or when started the shell.

### **Option 3: DigitalOcean cloud + Docker Installed**

1. Getting to the point where Docker installed
    * Follow the [Quick Start Guide - Digital Ocean CentOS 7 Server with Docker](https://github.com/glennleekg/readme/blob/master/qsg-docean-centos7-docker.md) to setup three droplets named node1, node2, and node3. 
    * Nodes must be created with Docker installed, and with specific ports of Swarm opened.

2. Make sure the node is working 
    * To access node1, run  
      ```
      $ ssh user@node1
      ... 
      (remotely log in into node1 with remote terminal access)
      ```
    * To test node1, run 
      ```
      $ docker info
      ... 
      (output show the docker information)
      
      $ ping node2
      ... 
      (output show successful ping log)
      ```
      similarly, access and test both node2 and node3

Reference:
[Docker Swarm Firewall Ports](https://www.bretfisher.com/docker-swarm-firewall-ports/)

## _**Step 2 ~ Create Docker Swarm**_

Open three terminal windows with access to node1, node2, and node3 respectively.

### **node1: initialize as manager node**
1. Go to node1 terminal
  
    To initialize Swarm, run
    ```
    $ docker swarm init
    Error response from daemon: could not choose an IP address to advertise since this system has multiple addresses on different interfaces (192.168.0.18 on eth0 and 172.18.0.58 on eth1) - specify one with --advertise-addr
    ```
    Note:  
    192.168.0.18 is the public IP address  
    172.18.0.58 is the private IP address

    The command want us to specify an IP address to advertise the Swarm service on, the IP address need to be accessible from the other nodes. In this case, use the public IP address, run
    ```
    $ docker swarm init --advertise-addr 192.168.0.18
    Swarm initialized: current node (xe5ibu8eyon0q5qajlpbcmh04) is now a manager.

    To add a worker to this swarm, run the following command:

        docker swarm join --token SWMTKN-1-04siscjef9ipeikprp08s9v6ph411oixv2qyf4pu8akkk14koj-ep445hzj45v2jdbhu2aykscfj 192.168.0.18:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    ```

2. At node1 terminal

    To list nodes in the swarm, use ```docker node ls``` command. For the first time Swarm initialization, it will show one manager node is created, it is marked as leader, since we can only have one leader at a time amongst all managers.
    ```
    $ docker node ls
    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
    xe5ibu8eyon0q5qajlpbcmh04 *   node1               Ready               Active              Leader              18.09.3
    ```

### **node2: Join as worker node, then update to manager node**
3. Go to node2 terminal

    To join as worker node, copy and ```docker swarm join``` command in node1 terminal generated by ```docker swarm init```, paste it here and run it
    ```
    $ docker swarm join --token SWMTKN-1-04siscjef9ipeikprp08s9v6ph411oixv2qyf4pu8akkk14koj-ep445hzj45v2jdbhu2aykscfj 192.168.0.18:2377
    This node joined a swarm as a worker.
    ```

4. Go to node1 terminal

    To list nodes in the swarm, use ```docker node ls``` command. Now it show two nodes in total, notice that node2 is created as worker node, it has no manager status.
    ```
    $ docker node ls
    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
    xe5ibu8eyon0q5qajlpbcmh04 *   node1               Ready               Active              Leader              18.09.3
    n2p1dv8o1a9oslrx8htcze6zk     node2               Ready               Active                                  18.09.3
    ```

5. Go to node2 terminal

    Notice that worker node can't be used to view or modify cluster state, it can't use any Swarm commands, error responded from daemon as follows. 
    ```
    $ docker node ls
    Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.

    $ docker service ls
    Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.
    ```

6. Go to node1 terminal

    To promote node2 to manager, run
    ```
    $ docker node update --role manager node2
    node2
    ```

7. At node1 terminal

    To list nodes in the swarm, use ```docker node ls``` command. Now node2 is on "Reachable" manager status, while the node1 is still on "Leader" manager status.
    ```
    $ docker node ls
    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
    xe5ibu8eyon0q5qajlpbcmh04 *   node1               Ready               Active              Leader         18.09.3
    n2p1dv8o1a9oslrx8htcze6zk     node2               Ready               Active              Reachable         18.09.3
    ```
    Note:
    The little asterisk indicates the node you are currently talking to.

### **node3: Join as manager node**
8. Go to node1 terminal

    To generate token for other node to join as manager node, run
    ```
    $ docker swarm join-token manager
    To add a manager to this swarm, run the following command:

        docker swarm join --token SWMTKN-1-04siscjef9ipeikprp08s9v6ph411oixv2qyf4pu8akkk14koj-c4h3e05g593ivmjpnue1lmvnz 192.168.0.18:2377
    ```

9. Go to node3 terminal

    To join as manager node, copy the ```docker swarm join``` command in node1 terminal generated by ```docker swarm join-token manager```, paste it here and run it
    ```
    $ docker swarm join --token SWMTKN-1-04siscjef9ipeikprp08s9v6ph411oixv2qyf4pu8akkk14koj-c4h3e05g593ivmjpnue1lmvnz 192.168.0.18:2377
    This node joined a swarm as a manager.
    ```

10. Go to node1 terminal
  
    To list nodes in the swarm, use ```docker node ls``` command. Now it show three nodes in total, notice that node3 is created as manager node, and it is on "Reachable" manager status. They all have manager status indicating that they are managers.
    ```
    $ docker node ls
    ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
    xe5ibu8eyon0q5qajlpbcmh04 *   node1               Ready               Active              Leader              18.09.3
    n2p1dv8o1a9oslrx8htcze6zk     node2               Ready               Active              Reachable           18.09.3
    w6apm3i6zonuuyps294hbx4ty     node3               Ready               Active              Reachable           18.09.3
    ```


### _Token generated for other node to join the Swarm_
* You can get these tokens at any time, you do not need to write them down or save them, they are part of the swarm configuration and stored encrypted on disk.
* You can change these tokens in case they get exposed or you maybe getting a server that have had vulnerability on it. You can actually rotate those keys where you want to make sure that no nodes can join the swarm using the old key.

### **service: start a 3 replicas service**
Now we have a 3-node, redundant swarm, with three managers. Once you have got the swarm created, you  don't need to be typing commands into all the different nodes, you can really operate the whole swarm, for most things, from node1.

11. Go to node1 terminal

    To create a 3 replicas service, run
    ```
    $ docker service create --replicas 3 alpine ping 8.8.8.8
    oe9f7oqoy90f0q3v6a1f72n7z
    overall progress: 0 out of 3 tasks
    overall progress: 3 out of 3 tasks
    1/3: running
    2/3: running
    3/3: running
    verify: Service converged
    ```
    
    To see the list of services in the swarm, use ```docker service ls``` command. The output shows one service listed, given a random name (i.e. zen_mclaren) since we didn't specified, and have 3/3 replica spun up.
    ```
    $ docker service ls
    ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
    oe9f7oqoy90f        zen_mclaren         replicated          3/3                 alpine:latest
    ```

    To see the current node running which tasks or containers, use ```docker node ps``` command. The output shows node1 is running the zen_mclaren.3 task or container.
    ```
    $ docker node ps
    ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
    iiag2rrwer99        zen_mclaren.3       alpine:latest       node1               Running             Running 24 minutes ago
    ```

    To see a specified node running which tasks or containers, use ```docker node ps <node_name>``` command. The output shows node2 is running the zen_mclaren.1 task or container, where node3 is running the zen_mclaren.2 task or container.
    ```
    $ docker node ps node2
    ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
    u3ggkkctmpm3        zen_mclaren.1       alpine:latest       node2               Running             Running 28 minutes ago

    $ docker node ps node3
    ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
    7xllgir0uj5f        zen_mclaren.2       alpine:latest       node3               Running             Running 29 minutes ago
    ```

    To see the full list of tasks or containers of the service, use ```docker service ps <service_name>``` command. The output shows full list, and which task or container is running on which node.
    ``` 
    $ docker service ps zen_mclaren
    ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
    u3ggkkctmpm3        zen_mclaren.1       alpine:latest       node2               Running             Running 31 minutes ago
    7xllgir0uj5f        zen_mclaren.2       alpine:latest       node3               Running             Running 31 minutes ago
    iiag2rrwer99        zen_mclaren.3       alpine:latest       node1               Running             Running 31 minutes ago
    ```
    