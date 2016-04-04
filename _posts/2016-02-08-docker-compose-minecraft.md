---
layout: post
title: Using Docker Compose to create and run Minecraft servers
categories: [technology, gaming]
tags: [docker, docker-compose, minecraft, spigot, bungeecord]
description: Running two or more Minecraft servers becomes simplified with Docker Compose, Spigot, and Bungeecord.
comments: true
---

Running two or more Minecraft servers becomes simplified with Docker Compose, Spigot, and Bungeecord!

## A coming together of interests

My kids introduced me to Minecraft a couple years back, and it really struck a nerve that propelled into borderline obsession.  I setup a family SMP (aka Survival Multiplayer) server, and began lots of small projects with Minecraft at the center.

Around that time containers and Docker were shooting up the technology radar charts, and I began experimenting with Minecraft vanilla containers then on to Bukkit then finally to Spigot and Bungeecord.  I forked the image [nimmis/docker-spigot](https://github.com/nimmis/docker-spigot), and converted it to [Bungeecord image](https://github.com/rehf27/docker-bungeecord).  I tested it out, and uploaded to Docker Hub.

## Someone picks up the baton

The author of [Rob's Ramblings](http://blog.irrelevant.com/2015/03/minecraft-spigot-bungeecord-and-docker.html) picked up where I left off, and detailed the setup needed to leverage data containers for greater portability and resuse.  After having Rob's post sit in my [Pocket](https://getpocket.com/) list for too long, I finally revisited Docker, Data Containers, and Minecraft.

## Using Docker Compose for Bungeecord + Spigot Minecraft server containers

I expanding on Rob's work by using docker-compose to declaratively setup the configuration he detailed.  I also added in some handy scripts to accelerate some of the extra steps needed to get the Spigot servers up and running.

In the future, I'd like to create a Yeoman generator or something similar to automatically configure the data containers and docker-compose.yml file.

But first...here is the [docker-compose.yml](https://github.com/rehf27/docker-bungeecord/blob/master/docker-compose.yml) for Rob's setup.

### Docker, docker-compose and docker-machine

For information on using docker, docker-machine or docker-compose...

- [Docker](https://docs.docker.com/engine/understanding-docker/)
- [Docker Compose](https://docs.docker.com/compose/overview/)
- [Docker Machine](https://docs.docker.com/machine/overview/)

In the directory containing the docker-compose.yml file, run from the terminal...

```
docker-compose up
```

This command will download the images, create the data and server containers, and run them.

### Additional steps

Update each Minecraft server's server.properties to disable online-mode, which will force players through the Bungeecord server for access.

- To connect to the shell of a data container

```
docker run --rm -t -i --volumes-from $CONTAINER_NAME centos /bin/bash
```

1. World 0

```
docker run --rm -t -i \
--volumes-from dockerbungeecord_world0data_1 \
centos \
sed -i 's/online-mode=true/online-mode=false/' /minecraft/server.properties
```
2. World 1

```
docker run --rm -t -i \
--volumes-from dockerbungeecord_world1data_1 \
centos \
sed -i 's/online-mode=true/online-mode=false/' /minecraft/server.properties
```

3. World 2

```
docker run --rm -t -i \
--volumes-from dockerbungeecord_world2data_1 \
centos \
sed -i 's/online-mode=true/online-mode=false/' /minecraft/server.properties
```

### For one Minecraft server

1. Connect to the data container, and run a search and replace command.

```
docker run --rm -t -i \
--volumes-from dockerbungeecord_bungeecorddata_1 \
centos /bin/bash \
sed -i 's/localhost:25565/world0server:25565/' /bungeecord/config.yml
```

### For multiple Minecraft servers
1. Connect to the data container's bash shell

```
docker run --rm -t -i \
--volumes-from dockerbungeecord_bungeecorddata_1 \
centos \
/bin/bash
```

2. Now in the data container's shell, run

```
cd bungeecord
vi config.yml
```

3. Replace the existing servers section with something like this...

```
servers:
  lobby:
    motd: '&1Lobby Server'
    address: world0server:25565
    restricted: false
  newworld:
    motd: '&1A New World'
    address: world1server:25565
    restricted: false
  anotherworld:
    motd: '&1Another World'
    address: world2server:25565
    restricted: false
```
