# Introduction
Docker Registry is an powerful app that manages storing and delivering Docker container images. Recently, docker images are widely tools for deploying an application. For example, if we want to install dependencies and packages separately to use Docker, teams can download a compressed image from a registry that contains all of the necessary components in case Devops. Moreover, developers can use CI/CD pushing images to a registry using continuous integration tools, such as Gitlab CI/CD or Jenkins or other CI/CD, to update images during production and development.

# Overview
This implementation presents to topology or environments of docker private registry by using server in the Digitalocean. In detail, server is installed Ubuntu 20.04 and docker engine, therefore, the service will provide the docker registry on port TCP/5000 and front-end by using NGINX on port 443/80 (Secure SSL by CERT-BOT, Let's Encrypt ) shown as figure below:

![GitHub Logo](/images/Network_topology.jpg)


# Prerequisites
Before you begin this guide, you’ll need the following:
* Ubuntu 20.04 servers set up by following the Ubuntu 20.04 initial server setup guide. you can use cloud Digital Ocean or AWS and so on.

* Docker and Docker-Compose installed on both servers by following the How to Install Docker-Compose on Ubuntu 20.04 tutorial. You only need to complete the first step of this tutorial to install Docker Compose. This tutorial explains how to install Docker as part of its prerequisites.  
