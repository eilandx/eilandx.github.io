---
title: Launch Minecraft server using Docker container
date: 2024-02-12
categories: [Guide, Tutorial ]
tags: [docker, minecraft, gaming]     # TAG names should always be lowercase
author: CrimsonCloak
pin: true
---

In this guide, we will set up a simple Minecraft server using a Docker container on a Linux machine (Ubuntu 22.04) We will use a pre-made image, spin it up using `docker run`, manage it using `docker compose`, learn how to customize it and playtest it. 



# Using Docker to launch a Minecraft server

- [Using Docker to launch a Minecraft server](#using-docker-to-launch-a-minecraft-server)
  - [Prerequisites](#prerequisites)
  - [Docker](#docker)
    - [Docker image](#docker-image)
    - [Docker container](#docker-container)
    - [Docker compose file](#docker-compose-file)
    - [Joining your Minecraft server](#joining-your-minecraft-server)
    - [Managing the Docker container](#managing-the-docker-container)
      - [Starting the container](#starting-the-container)
      - [Stopping the container](#stopping-the-container)
      - [Restarting the container](#restarting-the-container)
    - [Customizing your Minecraft server](#customizing-your-minecraft-server)


## Prerequisites
We need to make sure that Docker and all its requirements are installed on the machine that will be hosting our server. This could be your own laptop, a virtual machine or a dedicated server. For documentation on installing Docker, please refer to the official [Docker documentation](https://docs.docker.com/engine/install/). We will be hosting the Minecraft server on Ubuntu 22.04, but the steps should be similar for other distributions. 

> When Docker is installed and running, you can add your user to the docker group to avoid having to use sudo for every docker command - `sudo usermod -aG docker $USER`. You will need to log out and back in for this to take effect, and this is not recommended in production environments. 
{: .prompt-tip }

## Docker

Docker will run our server in what is called a 'container' - a (semi-) isolated environment that runs on top of the host operating system. Docker is great because the containers can be easily started, transferred or stopped on any machine that has the Docker Engine installed. Containers are based on "images" - you can think of an image as a template or recipe for a container that contains all the required software and requirements. We will be using a pre-built Docker image to host our server, as creating your own images is beyond the scope of this little guide.

### Docker image

For this simple guide we will use a Docker image (~ template for container) that is built and supported by the community. We will be using itzg's Docker image for this demonstration. The image can be found [here](https://github.com/itzg/docker-minecraft-server) - full credit to [itzg](https://github.com/itzg) for creating and maintaining this wonderful image.


### Docker container

In order to start our Minecraft container with the specific image, we can use the following simple docker run command. 
```console  
docker run -p 25565:25565 -v /srv/minecraft/data:/data -i -t --restart unless-stopped itzg/minecraft-server
```


This command will start the Minecraft server and map the server's data to a folder on the host machine - in this case "/srv/minecraft". This way, the server's data will be persistent, accessible from the server and can be backed up easily. It will also open the server's port 25565 (the standard Minecraft port) to the host machine, so that the server can be accessed on the host.

> `docker run` is a simple command-line tool for managing your Docker containers! It is ideal for getting started and experimenting with simple containers.
{: .prompt-tip }

### Docker compose file 

Docker run commands are very nice and all that, but they can get very complicated and when managing multiple Docker containers, it can get quite overwhelming. Docker compose is a tool to mitigate this problem. A docker compose file is a simple yaml file that can be used to define and run Docker applications. While this is often used for multi-container apps or setups, the benefit to using it for our simple container is that it makes our setup more readable and maintainable.

We can create a simple docker-compose file which contains the same parameters and information as our docker run command. We can place this `docker-compose.yml` file wherever we want in its own folder, for example "/srv/docker/minecraft". 
    
```console
version: "3.8"

services:
    minecraft-server:
        image: itzg/minecraft-server
        container_name: minecraft-server
        environment:
            EULA: "true"
        ports:
            - "25565:25565"
        volumes:
            - /srv/minecraft/data:/data
        stdin_open: true
        tty: true
        restart: unless-stopped
```
{: file='docker-compose.yml'} 


> Please be aware that you can only have a single docker-compose.yml file per directory! If you want to run multiple containers, you can organize them in their own directories or you can define them all in the same file if they work together.
{: .prompt-warning }

We can then start our Minecraft server by using the following command in the directory containing our compose file:

```console
docker compose up -d
```

### Joining your Minecraft server
Congratulations! You can now join your Minecraft server on the host machine on [IP]:25565. Give it a minute to start up and then join using your Minecraft client!

> If your server is not in your local network, you can not access it through the internet. For that to be possible, you would need to perform a port-forward. For the purposes of this demo, we assume that you are simply hosting a server for your friends or family to join on your local network or for testing purposes.
{: .prompt-warning }


### Managing the Docker container

Below are some simple commands to manage your Docker container(s). You can use either `docker run` or `docker compose` commands, depending on your preference. We would advise using `docker compose` for its readability and maintainability - but remember that this requires you to perform the command(s) in the correct directory!

#### Starting the container
```console
docker compose up -d
```
{: .language-bash}

This command will start the container in the background. The `-d` flag will make the process run in the background - and not lock your terminal.

#### Stopping the container
```console  
docker compose down
```
{: .language-bash}

This command will stop the container and remove it. The container's data will not be removed when you use the volume binding, so you can start it again later without losing data.

#### Restarting the container
```console
docker compose restart
```
{: .language-bash}

This command will restart the container. This is useful if you have made changes to the server's settings or if you have updated the server's image.





### Customizing your Minecraft server

After testing your Minecraft server, you might want to customize it. You can do this by modifying the server.properties file in the /srv/minecraft/data folder. This file contains all the settings for your Minecraft server. You can also install mods, plugins and resource packs by placing them in the /srv/minecraft/data folder. 

If you want to customize things such as the Minecraft version, you can do so by [adding environment variables](https://docker-minecraft-server.readthedocs.io/en/latest/versions/minecraft/) to your docker-compose file. There are many things you can customise with these environment variables, and the project has [great documentation](https://docker-minecraft-server.readthedocs.io/en/latest/configuration/server-properties/) you can refer to. Changing the minecraft version, for example, can be done like this:
```console
version: "3.8"

services:
    minecraft-server:
        image: itzg/minecraft-server
        container_name: minecraft-server
        environment:
            EULA: "true"
            VERSION: "1.18.1"
        ports:
            - "25565:25565"
        volumes:
            - /srv/minecraft/data:/data
        stdin_open: true
        tty: true
        restart: unless-stopped
```
{: file='docker-compose.yml'} 


After adding the environment variables to your docker-compose.yml file, be sure to restart your server with the following command in the docker-compose.yml directory:
```console
docker-compose restart
```

There are many ways to customize your server to your liking, but that is beyond the scope of this guide. This tutorial is meant to get you started - the rest of the customization, playtesting and fun is up to you!




