---
title: "SNAS Collector Docker Install"
date: 2017-05-07T15:04:10.000Z
---

SNAS Collector Docker Install Steps
===================================

<!--more-->

Collector container is used for collecting BMP messages from BMP senders, e.g. routers. This container can be distributed.

### Container Includes:
* **Openbmpd** - Latest collector (listening port is TCP 5000)

### Recommended Current Linux Distributions:

  1. Ubuntu 16.04/Xenial
  1. CentOS 7/RHEL 7

### **Installation Steps**

- - -

### 1) Install docker
Docker host should be **Linux x86_64**.   Follow the [Docker Instructions](https://docs.docker.com/installation/) to install docker.  

- - -

### 2) Download the docker image

    docker pull openbmp/collector

- - -

### 3) Add persistent configs [OPTIONAL] 


#### On host create persistent config location:

    mkdir -p /var/openbmp/config
    chmod 777 /var/openbmp/config

#### config/hosts
You can add custom host entries so that the collector will reverse lookup IP addresses
using a persistent hosts file.

Run docker with ```-v /var/openbmp/config:/config``` to make use of the persistent config files.

You can also add other hosts into a container’s /etc/hosts file by using one or more --add-host flags. 

#### config/openbmpd.conf
You can provide a customized **openbmpd.conf**.  See [Config Example](https://github.com/OpenBMP/openbmp/blob/master/Server/openbmpd.conf) in github.


### 4) Run docker container

#### Environment Variables
Below table lists the environment variables that can be used with ``docker run -e <name=value>``

NAME | Value | Details
:---- | ----- |:-------
**KAFKA\_FQDN** | hostname | **REQUIRED**. Fully qualified hostname for the docker host of this container. Will be used for API and Kafka. It is also the default OPENBMP_ADMIN_ID.
OPENBMP\_ADMIN\_ID | name or IP | Name or IP of the collector, default is the docker hostname.
OPENBMP\_BUFFER | Size in MB | Defines the openbmpd buffer per router for BMP messages. Default is 16 MB.

#### Run normally

- - -

### **IMPORTANT:** You **MUST define the KAFKA_FQDN** as a **'hostname'** (or fqdn) and not by IP. 
#### If all containers are running on the same node, this hostname can be local specific, such as 'localhost' or 'myhost'. 
#### If Kafka is running on a different server than the consumers and producers, then the KAFKA_FQDN should be a valid hostname that can be resolved using DNS. This can be internal DNS or manually done by updating the /etc/hosts file on each machine.

- - -

    docker run -d --name=openbmp_collector -e KAFKA_FQDN=localhost \
         -v /var/openbmp/config:/config \
         -p 5000:5000 \
         openbmp/collector


### **Monitoring/Troubleshooting**
You can navigate all the log files from within the container. Connect to container using:
    
    docker exec -it openbmp_collector bash

Or, you can use standard docker exec commands on host to monitor the log files.  To monitor collector, use: 

    docker exec openbmp_collector tail -f /var/log/openbmpd.log

Or, you can monitor the docker container by getting the console logs. This is useful if the container exits due to invalid start or for another reason. 
To see console logs for collector, use:

    docker logs openbmp_collector
    

### **System Start/Restart Config (Ubuntu 16.04/Xenial)**
By default, the containers will not start automatically on system boot/startup.  You can use the below example to instruct the openbmp/aio container to start automatically. 

You can read more at [Docker Admin Guide](https://docs.docker.com/engine/admin/start-containers-automatically/) on how to start containers automatically. 
You can use the below example to instruct the container to start automatically.

> #### IMPORTANT
> The ```--name=openbmp_collector``` parameter given to the ```docker run``` command is used with the ```-a openbmp_collector``` parameter below to start the container by name instead of container ID.  You can use whatever name you want, but make sure to use the same name used in docker run.

    cat <<END > /etc/init/collector-openbmp.conf
    description "SNAS Collector container"
    author "tim@openbmp.org"
    start on filesystem and started docker
    stop on runlevel [!2345]
    respawn
    script
      /usr/bin/docker start -a openbmp_collector
    end script
    END
     
     


