# Quick Start Guide - Development Environment On Docker 
# 1. Setup Docker CE on Ubuntu
References: Get Docker CE for 
[Centos](https://docs.docker.com/install/linux/docker-ce/centos/),
[Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/),
[MacOS](https://docs.docker.com/docker-for-mac/install/),
[Windows](https://docs.docker.com/docker-for-windows/install/).

## _**Step 1 ~ Install using the convenience script**_
Note: Using these scripts is not recommended for production environments.

### **Option A: latest stable release**
The command uses the script at get.docker.com to install the latest release of Docker CE on Linux. 
```
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```
### **Option B: edge test release**
The command uses the script at test.docker.com to install the edge release of Docker CE on Linux.
```
$ curl -fsSL https://test.docker.com -o test-docker.sh
$ sudo sh test-docker.sh
```
## _**Step 2 ~ Use Docker as non-root user**_
If you would like to use Docker as a non-root user, you should now consider adding your user to the “docker” group with something like:

[Warning!!! Docker daemon attack surface.](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface)

```
$ sudo usermod -aG docker your-user
```
## _**Optional Linux Post-installation Steps**_
https://docs.docker.com/install/linux/linux-postinstall/

# 2. Create and Run Standalone MariaDB Container
>_This section is using "docker container run" command to create a standalone MariaDB Container from official image. For building own private custom made image used for creating containers, defining and running multiple containers, please refer to [3. Define, Create, and Run Multiple Containers](#3.-Define,-Create-and-Run-Multiple-Containers)._

Reference: [Docker Official's Image](https://hub.docker.com/_/mariadb), [MariaDB Official's Image](https://hub.docker.com/r/mariadb/server)

Documentation: [Docker Official's Documentation](https://docs.docker.com/samples/library/mariadb/), [MariaDB Official's Documentation](https://mariadb.com/kb/en/library/installing-and-using-mariadb-via-docker/)

## _**Step 1 ~ Download MariaDB Image**_
You can search and download MariaDB Images from [Official MariaDB Repository](https://hub.docker.com/u/mariadb/) or from [Docker Hub](https://hub.docker.com/search?q=mariadb&type=image) online, or you may use the command:
```
$ docker search mariadb
```
Pull the MariaDB Image that you have selected
```
# if you select the MariaDB Official Image version 10.3         
$ docker pull mariadb/server:10.3

# if you select the Docker Official Image version 10.3
$ docker pull mariadb:10.3
```
Check if the MariaDB Image was pulled
```
$ docker images
```
Note: [MariaDB Official Image's DockerFile](https://github.com/mariadb-corporation/mariadb-server-docker) is **forked** from [Docker Official Image's DockerFile](https://github.com/docker-library/mariadb)

## _**Step 2 ~ Configure Storage and Network**_
Bear in mind that all the docker containers created here are using named volume and user-defined bridge network for better managing in the development environment.

### **For named volume,**
```
$ docker volume create mariadb-server-data

## docker container run command in the next step will use this flag
--mount type=volume,source=mariadb-server-data,target=/var/lib/mysql
```
Reference: [Manage data in Docker](https://docs.docker.com/storage/), by [using volumes](https://docs.docker.com/storage/volumes/), [using bind mounts](https://docs.docker.com/storage/bind-mounts/), and [using tmpfs mounts](https://docs.docker.com/storage/tmpfs/).

### **For user-defined bridge network,**
```
$ docker network create --driver bridge back-tier

## docker container run command in the next step will use this flag
--network back-tier
```
Reference: [Configure networking](https://docs.docker.com/network/), by [using bridge networks](https://docs.docker.com/network/bridge/), [using overlay networks](https://docs.docker.com/network/overlay/), [using host networking](https://docs.docker.com/network/host/), and etc.

## _**Step 3 ~ Run MariaDB Container**_
> You may skip Step 1 because "docker container run" command will pull the image if not found.

Note: Container created from MariaDB Image is running mysqld once started. (Tip for Deep Diver, please refer to the MariaDB Image's [Dockerfile](https://github.com/docker-library/mariadb/blob/master/Dockerfile.template), the CMD code at bottom of Dockerfile)

### **Option A: Start a MariaDB server instance with default configuration**
You may use this if the default configuration suit your needs, or if you prefer to add configuration option directly to mariadb-server container's [MariaDB configuration file](#Appendix:-MariaDB-Configuration-Files-in-container).
```
$ docker container run --name mariadb-server --network back-tier \
  --mount type=volume,source=mariadb-server-data,target=/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=password -d mariadb/server:10.3
```

### **Option B: Start a MariaDB server instance by using custom configuration file at host**
Using --mount to bind mount the /path/on/host/mariadb.conf.d directory at host to /etc/mysql/mariadb.conf.d directory in docker container. Please place your configuration files to the /path/on/host/mariadb.conf.d at host.

Note: If you mount a bind mount into a directory in the container in which some files or directories exist, these files or directories are obscured by the mount.
```
$ docker container run --name mariadb-server --network back-tier \
  --mount type=bind,source=/path/on/host/mariadb.conf.d,target=/etc/mysql/mariadb.conf.d \
  --mount type=volume,source=mariadb-server-data,target=/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=password -d mariadb/server:10.3
```

### **Option C: Start a MariaDB server instance by passing configuration options**
Passing in the configuration options to mysqld
```
$ docker container run --name mariadb-server --network back-tier \
  --mount type=volume,source=mariadb-server-data,target=/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=password -d mariadb/server:10.3 \
  --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

## _**Step 4a ~ Connect to MariaDB Server from the MySQL command line client**_
Note: You do not need to pass the configuration option --default-character-set=utf8mb4 if you have added it directly to mariadb-server container's [mariadb configuration file](#Appendix:-MariaDB-Configuration-Files-in-container).

### **Option A: Run the command in mariadb-server container**
```
$ docker container exec -i -t mariadb-server mysql -u root -p --default-character-set=utf8mb4
```

### **Option B: Connect from host**
Note: Your host machine need to install mysql client
```
# find the container's IP address
$ docker container inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mariadb-server

# run the client and set the server address ("-h") to the container's IP address found
$ mysql -h 172.17.0.2 -u root -p --default-character-set=utf8mb4 
```

## _**Step 4b ~ Connect to MariaDB Server from an application in another Docker container**_
Create the app container with --network flag which connect the web app container to back-tier network, thus some-app and mariadb-server can reach each other using the container name without knowing the IP address.
```
$ docker run --name some-app --network back-tier -d app-image
```
To connect a running container to an existing user-defined bridge, run:
```
$ docker network connect back-tier some-app
```
To disconnect a running container from a user-defined bridge, run:
```
$ docker network disconnect back-tier some-app
```

# 3. Define, Create and Run Multiple Containers
>_This section is using "docker image build" command to build own private custom made image used for creating container, and "docker-compose" command to run multiple containers which relationships are defined in a yml file. For creating and running a standalone MariaDB container, please refer to [2. Create and Run Standalone MariaDB Container](#2.-Create-and-Run-Standalone-MariaDB-Container)._

## _**Step 1 ~ Building Custom Made MariaDB Image from Dockerfile**_
> You may skip this step, Docker Compose in Step 3 handle this.

prepare the Dockerfile for MariaDB
```
FROM mariadb/server:10.3

RUN \
  touch /etc/mysql/mariadb.conf.d/charset.cnf \
  && printf " \
  [client]\n \
  default-character-set = utf8mb4\n \
  \n \
  [mysqld]\n \
  character-set-server  = utf8mb4\n \
  collation-server      = utf8mb4_unicode_ci\n \
  character_set_server  = utf8mb4\n \
  collation_server      = utf8mb4_unicode_ci\n" \
  >> /etc/mysql/mariadb.conf.d/charset.cnf

CMD ["mysqld"]
```

build the MariaDB Image
```
$ docker image build -t glennleekg/mariadb-server-utf8mb4:10.3 .
```

## _**Step 2 ~ Building Custom Made OpenjdkSbt Image from Dockerfile**_
> You may skip this step, Docker Compose in Step 3 handle this.

prepare the Dockerfile for OpenjdkSbt
```
FROM openjdk:8-jdk-alpine

ENV SBT_VERSION 1.2.8

RUN \
  apk add --no-cache bash && \
  apk add --no-cache --virtual=build-dependencies curl gzip && \
  curl -fsSL https://github.com/sbt/sbt/releases/download/v$SBT_VERSION/sbt-$SBT_VERSION.tgz | tar -xzf - -C /usr/local && \
  ln -s /usr/local/sbt/bin/* /usr/local/bin/ && \
  apk del build-dependencies

ENV PROJECT_PATH /projects

WORKDIR $PROJECT_PATH

CMD ["bash"]
```

build the OpenjdkSbt Image
```
$ docker image build -t glennleekg/openjdk-sbt:8-jdk-alpine
```

## _**Step 3 ~ Composing Containers from compose.yml**_

### **Prepare the code for web app example**
```
$ mkdir /your/path/to/projects
$ cd /your/path/to/projects
$ git clone https://github.com/glennleekg/play-scala-anorm-example.git
$ cd /your/path/to/projects/play-scala-anorm-example 
$ git checkout 2.6.x-mariadb
```
### **Composing Containers from docker-compose.yml for Developement Environment**
> You may skip this, docker-compose.yml is included in the git repo

prepare the docker-compose.yml in the /your/path/to/projects/play-scala-anorm-example directory
```
version: '3.7'

services:
  mariadb-server:
    build:
      context: https://raw.githubusercontent.com/glennleekg/mariadb-server-utf8mb4/master/10.3/Dockerfile
    image: glennleekg/mariadb-server-utf8mb4:10.3
    environment: 
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: playdb
    volumes:
      - mariadb-server-data:/var/lib/mysql
    networks:
      - backend
    
  play-scala-anorm:
    build:
      context: https://raw.githubusercontent.com/glennleekg/openjdk-sbt/master/11-jdk-slim-dev/Dockerfile
    image: glennleekg/openjdk-sbt:8-jdk-alpine
    expose:
      - 9000
      - 9443
    ports:
      - 9000:9000
      - 9443:9443
    volumes:
      - ${PWD}:/root/play-scala-anorm-example
      - ~/.ivy2:/root/.ivy2
      - ~/.sbt:/root/.sbt
    networks:
      - backend
    working_dir: /root/play-scala-anorm-example
    stdin_open: true
    tty: true

volumes:
  mariadb-server-data:
    driver: local

networks:
  backend:
    driver: bridge

```

### **Compose development environment**
```
$ docker-compose up --build

# confirm all containers up and running
$ docker-compose ps
```

### **Accessing the container**
```
# to access play-scala-anorm
$ docker-compose exec play-scala-anorm bash

# to access play-scala-anorm with sbt shell or sbt ~run
$ docker-compose exec play-scala-anorm sbt shell
$ docker-compose exec play-scala-anorm sbt "~run"

# to access mariadb-server
$ docker-compose exec mariadb-server bash
```

### **Restart, Start, Stop, Pause, Unpause and Remove the container**
Basic Operations
``` 
$ docker-compose restart
$ docker-compose start
$ docker-compose stop
$ docker-compose pause
$ docker-compose unpause
```

stop and remove containers, networks, images, and volumes
```
# remove the containers without destroying the volume
$ docker-compose down

# remove the containers and the volume
$ docker-compose down -v
```

# Appendix: Basic Container Usage
### **Accessing the container**
To access the container via Bash, run:
```
$ docker container exec -it mariadb-server bash
```

### **Troubleshooting the container**
To view the container's logs, run:
```
$ docker container logs mariadb-server
```

### **Restart, Start, Stop, Pause, Unpause and Remove the container**
Basic Operations
```
$ docker container restart mariadb-server

$ docker container start mariadb-server

# SIGTEM terminate the process gracefully
$ docker container stop mariadb-server

# SIGKILL kill the process after timeout
$ docker container stop --time=30 mariadb-server

# SIGKILL kill the process immediately
$ docker container kill mariadb-server
```
Pause and Unpause Container
```
$ docker container pause node1 node2 node3

$ docker container unpause node1 node2 node3
```

Remove Container
```
# remove the container without destroying the volume for /var/lib/mysql
$ docker container rm mariadb-server

# remove the container and the volume for /var/lib/mysql
$ docker container rm -v mariadb-server
```

### **Creating database dumps**
```
$ docker exec mariadb-server sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /path/on/host/all-databases.sql
```

# Appendix: MariaDB configuration files in container
## _**MariaDB Configuration File Structure**_
- /etc/mysql/my.cnf 
  
  At the bottom of the file, there are two lines which include the file /etc/mysql/mariadb.cnf and the directory /etc/mysql/conf.d

- /etc/mysql/conf.d
  
  Place to add configuration file with option that overwrite the my.cnf

- /etc/mysql/mariadb.cnf
  
  At the bottom of the file, there are two lines which include the file /etc/mysql/mariadb.cnf and the directory /etc/mysql/conf.d

- /etc/mysal/mariadb.conf.d
  
  Place to add configuration file with option that overwrite the mariadb.cnf

## _**Edit the MariaDB configuration file in the mariadb-server container's**_
1. Access the container via Bash, run:

   ```
   $ docker exec -it mariadb-server bash
   ```

2. In the mariadb-server container, run:

   ```
   root@4d7a255d2833: vi /etc/mysql/mariadb.cnf
   ```

3. Add the configuration options
   
   ```
   [client]
   default-character-set = utf8mb4

   [mysqld]
   character-set-server  = utf8mb4
   collation-server      = utf8mb4_unicode_ci
   character_set_server  = utf8mb4
   collation_server      = utf8mb4_unicode_ci
   ```

## _**mysqld Options**_
To see a complete list of available options, run:
```
$ docker run -it --rm mariadb/server:10.3 --verbose --help
```
Reference: [mysqld Options](https://mariadb.com/kb/en/library/mysqld-options/)