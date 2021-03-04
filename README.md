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


* Update the apt package index and install packages to allow apt to use a repository over HTTPS:

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

* Use the following command to set up the stable repository.

```
$ echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```

* Install Docker engine

```
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io

```

* Verify that Docker Engine is installed correctly by running the hello-world image.

```
$ docker ps
$ sudo docker run hello-world
```

* Run this command to download the current stable release of Docker Compose:
```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.28.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

* Apply executable permissions to the binary:

```
$ sudo chmod +x /usr/local/bin/docker-compose
```
* Test the  docker-compose installation.

```
$ docker-compose --version
docker-compose version 1.28.5, build 1110ad01
```


# Step 2 — Setup Docker registry 

* Docker Registry is itself an application with multiple components, in this tutorial, we will use Docker Compose to manage the configuration. To start an instance of the registry, we need to set up a docker-compose.yml file to define the location where the registry will be storing its data.

```
$ mkdir ~/docker-private-registry && cd $_
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

* You can now start Docker Compose to check the setup:

```
docker-compose up
```

* You will see download bars in your output that show Docker downloading the Docker Registry image from Docker’s own registry :

```
Output of docker-compose up
Starting docker-registry_registry_1 ... done
Attaching to docker-registry_registry_1
registry_1  | time="2018-11-06T18:43:09Z" level=warning msg="No HTTP secret provided - generated random secret. This may cause problems with uploads if multiple registries are behind a load-balancer. To provide a shared secret, fill in http.secret in the configuration file or set the REGISTRY_HTTP_SECRET environment variable." go.version=go1.7.6 instance.id=c63483ee-7ad5-4205-9e28-3e809c843d42 version=v2.6.2
registry_1  | time="2018-11-06T18:43:09Z" level=info msg="redis not configured" go.version=go1.7.6 instance.id=c63483ee-7ad5-4205-9e28-3e809c843d42 version=v2.6.2
registry_1  | time="2018-11-06T18:43:09Z" level=info msg="Starting upload purge in 20m0s" go.version=go1.7.6 instance.id=c63483ee-7ad5-4205-9e28-3e809c843d42 version=v2.6.2
registry_1  | time="2018-11-06T18:43:09Z" level=info msg="using inmemory blob descriptor cache" go.version=go1.7.6 instance.id=c63483ee-7ad5-4205-9e28-3e809c843d42 version=v2.6.2
registry_1  | time="2018-11-06T18:43:09Z" level=info msg="listening on [::]:5000" go.version=go1.7.6 instance.id=c63483ee-7ad5-4205-9e28-3e809c843d42 version=v2.6.2

```

# Step 2 — Install Nginx 

* Nginx is available in Ubuntu’s default repositories, it is possible to install it from these repositories using the apt packaging system.
```
$ sudo apt update
$ sudo apt install nginx
```

* Adjusting the Firewall
```
$ sudo ufw app list
```
You should get a listing of the application profiles:

```
Output
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```
* As demonstrated by the output, there are three profiles available for Nginx:

Nginx Full: This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic)
Nginx HTTP: This profile opens only port 80 (normal, unencrypted web traffic)
Nginx HTTPS: This profile opens only port 443 (TLS/SSL encrypted traffic)

* You can enable this by typing:

```
$ sudo ufw allow 'Nginx HTTP'
```

* Checking web server

```
$ systemctl status nginx
```

* output should  be:

```
Output
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2020-04-20 16:08:19 UTC; 3 days ago
     Docs: man:nginx(8)
 Main PID: 2369 (nginx)
    Tasks: 2 (limit: 1153)
   Memory: 3.5M
   CGroup: /system.slice/nginx.service
           ├─2369 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─2380 nginx: worker process
```

* Finally, When you have your server’s IP address, enter it into your browser’s address bar:

```
http://your_server_ip
```
* You should receive the default Nginx landing page:
![GitHub Logo](https://assets.digitalocean.com/articles/nginx_1604/default_page.png)


# Step 3 — Setting Port Forwarding NGNIX
* before to make an implementation step, we need to config Https and register domain name for your server.
1. You need to have HTTPS (This case we use Certbot for Nginx ) : https://certbot.eff.org/
2.  Free Domian (free domain example: dockerdevops.tk ): https://www.freenom.com/en/freeandpaiddomains.html
set up on your Docker Registry server with Nginx, which means you can now set up port forwarding from Nginx to port 5000. Once you complete this step, you can access your registry directly at yourdomain.com.

* Open this file with your text editor:
```
sudo nano /etc/nginx/sites-available/example.com
```

* just fine  the existing location line. It will look like this: 
```
...
location / {
  ...
}
...
```

* You need to forward traffic to port 5000, where your registry will be running. You also want to append headers to the request to the registry, which provide additional information from the server with each request and response. Delete the contents of the location section, and add the following content into that section. Need to edit this file ` /etc/nginx/sites-available/yourdomain.com `


```
...
location / {
    # Do not allow connections from docker 1.5 and earlier
    # docker pre-1.6.0 did not properly set the user agent on ping, catch "Go *" user agents
    if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
      return 404;
    }

    proxy_pass                          http://localhost:5000;
    proxy_set_header  Host              $http_host;   # required for docker client's sake
    proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
    proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header  X-Forwarded-Proto $scheme;
    proxy_read_timeout                  900;
}
...
```

* Save and exit the file. Apply the changes by restarting Nginx:
```
sudo service nginx restart
```

* You can confirm that Nginx is forwarding traffic to port 5000 by running the registry:
```
cd ~/docker-registry
docker-compose up
```

* In a browser window, open up the following url:
```
https://example.com/v2
```
 
* You will see an empty JSON object, or:
```
{}
```
# Step 3 — Setting Up Authentication
With Nginx proxying requests properly, you can now secure your registry with HTTP authentication to manage who has access to your Docker Registry. To achieve this, you’ll create an authentication file with htpasswd and add users to it. HTTP authentication is quick to set up and secure over a HTTPS connection, which is what the registry will use.

* You can install the htpasswd package by running the following:
```
sudo apt install apache2-utils
```
* Now you’ll create the directory where you’ll store our authentication credentials, and change into that directory. $_ expands to the last argument of the previous command, in this case ~/docker-registry/auth:

```
mkdir ~/docker-private-registry/auth && cd $_
```

* Next, you will create the first user as follows, replacing username with the username you want to use. The -B flag specifies bcrypt encryption, which is more secure than the default encryption. Enter the password when prompted:

```
htpasswd -Bc registry.password username
```

Note: To add more users, re-run the previous command without the -c option, (the c is for create):

```
htpasswd registry.password username
```

* You can add environment variables and a volume for the auth/ directory that you created, by editing the docker-compose.yml file to tell Docker how you want to authenticate users. Add the following highlighted content to the file: `docker-compose.yml`

```
version: '3'

services:
  registry:
    image: registry:2
    ports:
    - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - ./auth:/auth
      - ./data:/data
```

* then 
```
docker-compose up -d
```

# Step 4 — Increasing File Upload Size for Nginx

* Before you can push an image to the registry, you need to ensure that your registry will be able to handle large file uploads. Although Docker splits large image uploads into separate layers, they can sometimes be over 1GB. By default, Nginx has a limit of 1MB on file uploads, so you need to edit the configuration file for nginx and set the max file upload size to 2GB, location: `/etc/nginx/nginx.conf`

```
sudo nano /etc/nginx/nginx.conf
```
* Find the http section, and add the following line:
```
...
http {
        client_max_body_size 2000M;
        ...
}
...
```

# Step 5 —  Pulling From Your Private Docker Registry
You are now ready to publish an image to your private Docker Registry, but first you have to create an image. For this tutorial, you will create a simple image based on the ubuntu image from Docker Hub. Docker Hub is a publicly hosted registry, with many pre-configured images that can be leveraged to quickly Docker size applications. Using the ubuntu image, you will test pushing and pulling to your registry.

* Log in with the username and password you set up previously:
```
docker login https://yourdomain.com
```
* then login to the registry and test push/pull

![GitHub Logo](/images/example_login.png)


# Reference 
1. https://docs.docker.com/engine/install/ubuntu/
2. How To Set Up a Private Docker Registry on Ubuntu 18.04: https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-18-04
3. https://www.freenom.com/en/freeandpaiddomains.html
4. https://certbot.eff.org/
5. How To Install Nginx on Ubuntu 20.04: https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04
6. Install docker on Ubuntu: https://docs.docker.com/engine/install/ubuntu/
7. How To Install and Use Docker Compose on Ubuntu 20.04: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04
8. Install Docker Engine on Ubuntu: https://docs.docker.com/engine/install/ubuntu/
9. Install Docker compose: https://docs.docker.com/compose/install/ 