# Introduction
Docker Registry is an powerful app that manages storing and delivering Docker container images. Recently, docker images are widely tools for deploying an application. For example, if we want to install dependencies and packages separately to use Docker, teams can download a compressed image from a registry that contains all of the necessary components in case Devops. Moreover, developers can use CI/CD pushing images to a registry using continuous integration tools, such as Gitlab CI/CD or Jenkins or other CI/CD, to update images during production and development.

# Overview
This implementation presents to topology or environments of docker private registry by using server in the Digitalocean. In detail, server is installed Ubuntu 20.04 and docker engine, therefore, the service will provide the docker registry on port TCP/5000 and front-end by using NGINX on port 443/80 (Secure SSL by CERT-BOT, Let's Encrypt ) shown as figure below:

![GitHub Logo](/images/Network_topology.png)


# Prerequisites
Before you begin this guide, you’ll need the following:
* Ubuntu 20.04 servers set up by following the Ubuntu 20.04 initial server setup guide. you can use cloud Digital Ocean or AWS and so on.

* Docker and Docker-Compose installed on both servers by following the How to Install Docker-Compose on Ubuntu 20.04 tutorial. You only need to complete the first step of this tutorial to install Docker Compose. This tutorial explains how to install Docker as part of its prerequisites.  

* Nginx installed on to private Docker Registry server . Nginx secured with Let’s Encrypt on your server for the private Docker Registry, Make sure to redirect all traffic from HTTP to HTTPS 

* A domain name that resolves to the server you’re using for the private Docker Registry. you can use any domain names system.  

# Step 1 — Installing Docker Engine and Docker-compose 

Prepare docker engine
Update the apt package index and install packages to allow apt to use a repository over HTTPS:

```
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg

```
$ Add Docker’s official GPG key:

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

```

Use the following command to set up the stable repository.

```
$ echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

Install Docker engine

```
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io

```

Verify that Docker Engine is installed correctly by running the hello-world image.

```
$ docker ps
$ sudo docker run hello-world
```


Docker Registry is itself an application with multiple components, in this tutorial, we will use Docker Compose to manage the configuration. To start an instance of the registry, we need to set up a docker-compose.yml file to define the location where the registry will be storing its data.

```
$ mkdir ~/docker-registry && cd $_
$ mkdir data

```

* Use text editor to create the `docker-compose.yml` configuration file:

```
version: '3'

services:
  registry:
    image: registry:2
    ports:
    - "5000:5000"
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - ./data:/data

```