---
title: "Starting the System"
weight: 3
---

## Installation Complete. Prepare for Something Amazing!

Now that we've installed Docksal, let's get it running.

Run the following in your terminal:

``` bash
$ fin system start
```

### Behind the Scenes

When you run `fin system start` the first time, a few things are happening behind the scenes.

For starters, you are initializing the Docksal application. If this is your first time, it will begin by downloading all of the required images from the registry and spin up the services. Because Docksal shares some services across every project, these services are not reliant on having a project started.

The services we're starting up are:

* `docksal-ssh-agent`
* `docksal-dns`
* `docksal-vhost-proxy`

They are responsible for making sure that your application has SSH access, can route requests to the correct project, and can resolve the `*.docksal` domain locally.

### Examining our System

`fin sysinfo` is one of the most important commands and utilities we have at our disposal. It tells us almost everything we need to know about our system, including what might be going wrong with it.

Let's take a look at the output of `fin sysinfo` to see what information we have access to.

``` bash
$ ❯ fin sysinfo
███  OS
Darwin Mac OS X 10.14.6
Darwin DorfMBP 18.7.0 Darwin Kernel Version 18.7.0: Tue Aug 20 16:57:14 PDT 2019; root:xnu-4903.271.2~2/RELEASE_X86_64 x86_64

███  ENVIRONMENT
MODE : VirtualBox VM
DOCKER_HOST : tcp://192.168.64.100:2376
DOCKSAL_NFS_PATH : /Users/jamesflynn/git
NFS EXPORTS:

# <ds-nfs docksal
/Users/jamesflynn/git 192.168.64.1 192.168.64.100 -alldirs -maproot=0:0
# ds-nfs>

███  FIN
fin version: 1.86.2

███  DOCKER COMPOSE
EXPECTED VERSION: 1.23.2
docker-compose version 1.23.2, build 1110ad01
docker-py version: 3.6.0
CPython version: 3.6.6
OpenSSL version: OpenSSL 1.1.0h  27 Mar 2018

███  DOCKER
EXPECTED VERSION: 18.09.2

Client: Docker Engine - Community
 Version:           18.09.2
 API version:       1.39
 Go version:        go1.10.8
 Git commit:        6247962
 Built:             Sun Feb 10 04:12:39 2019
 OS/Arch:           darwin/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.2
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.6
  Git commit:       6247962
  Built:            Sun Feb 10 04:20:28 2019
  OS/Arch:          linux/amd64
  Experimental:     false

███  DOCKER MACHINE
EXPECTED VERSION: 0.16.1
docker-machine version 0.16.1, build cce350d7

NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER     ERRORS
docksal   *        virtualbox   Running   tcp://192.168.64.100:2376           v18.09.2

███  DOCKSAL: PROJECTS
project             STATUS                virtual host                                  project root

███  DOCKSAL: VIRTUAL HOSTS

███  DOCKSAL: DNS
Successfully requested http://dns-test.docksal

███  DOCKER: RUNNING CONTAINERS
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS                PORTS                                                    NAMES
f95a1d600dbe        docksal/ssh-agent:1.2       "docker-entrypoint.s…"   3 days ago          Up 3 days (healthy)                                                            docksal-ssh-agent
f5d581ad9436        docksal/dns:1.1             "docker-entrypoint.s…"   3 days ago          Up 3 days (healthy)   192.168.64.100:53->53/udp                                docksal-dns
de0e1729e381        docksal/vhost-proxy:1.5     "docker-entrypoint.s…"   3 days ago          Up 3 days (healthy)   192.168.64.100:80->80/tcp, 192.168.64.100:443->443/tcp   docksal-vhost-proxy

███  DOCKER: NETWORKS
NETWORK ID          NAME                   DRIVER              SCOPE
f0767638af16        _default               bridge              local
909d942fc925        bridge                 bridge              local
37afaa2911ed        host                   host                local
86dbe729b8fc        none                   null                local

███  VIRTUALBOX
EXPECTED VERSION: 5.2.26
5.2.26r128414

███  DOCKSAL MOUNTS

███  HDD Usage
Filesystem                Size      Used Available Use% Mounted on
/dev/sda1                46.1G      4.4G     39.3G  10% /mnt/sda1
```

Instead of going over each item in depth, we'll briefly cover the sections.

* **OS** - The current host machine OS and relevant information.
* **ENVIRONMENT** - Info about the current Docker system.
* **FIN** - The version of `fin` that we're using.
* **DOCKER COMPOSE** - Current version of Docker Compose
* **DOCKER** - Current version of Docker Engine and Docker's host and client.
* **DOCKER MACHINE** - Current version of Docker Machine.
* **DOCKSAL: PROJECTS** - A list of projects Docksal knows about.
* **DOCKSAL: VIRTUAL HOSTS** - A list of the virtual hosts Docksal is watching for.
* **DOCKSAL: DNS** - Output indicating whether or not the DNS service is working.
* **DOCKER: RUNNING CONTAINERS** - What containers/services are currently active.
* **DOCKER: NETWORKS** - The current networks Docksal is handling.
* **VIRTUALBOX** - The running version of VirtualBox (if applicable).
* **DOCKSAL MOUNTS** - Current mounted volumes.
* **HDD Usage** - How much disk storage Docksal is currently eating up.

This information is extremely useful when debugging a project, if something isn't working quite right, or if there are questions about your configurations.

### Summary

We've installed and started Docksal, checked our system, and explored how to inspect our system with a single command. Up next, we're going to spin up a project and explore how to work within our new environment.
