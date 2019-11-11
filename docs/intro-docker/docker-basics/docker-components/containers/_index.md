---
title: "Docker Containers"
weight: 4
---

## Docker Containers

In our [high level overview](/intro-docker/high-level/) we talked briefly about Linux containers, and that's what Docker is all about: Containers.

A container is a self-contained application that runs on a shared kernel. Docker containers are no different. When you run an image, you are creating a container that has followed all of the instructions in your Dockerfile to become an application. This can be something as small as a line of code that spouts out `Hello, world!` to a fully functional operating system with everything one may need to develop a complex application in a compiled language and test it.

Some things to note:

* A container is not permanent: If a container is removed, all data that is not in the image is destroyed.
* A container is portable: By sharing an image or a Dockerfile to build an image, a container can be recreated identically anywhere Docker can be found.
* A container is only as powerful as its host: It wouldn't be a good idea to try running a containerized version of a graphical operating system on a Raspberry Pi.
* A container is limited by what is in the image: There will be no more or no less functionality than what is in the Dockerfile or image definition.

In contrast to a Virtual Machine, a container is much smaller and easier to spin up, often in a matter of seconds.

### From Image to Container

In the previous section we talked about images and how they form the blueprints for containers by following the instructions within a Dockerfile or similar. Now we're going to run a few examples of spinning up containers.

Let's take the image we created in our last step and spin up a container. From within the `~/projects/docksal-training-docker/` folder we used in the last step we're going to now run the container.

Enter the following in your terminal:

``` bash
$ docker run -i -t \
  --name=test_container \
  image-example:1.0.0 /bin/bash
```

This will start the container and allow us to use the command line from within the container. Go ahead and poke around a bit. It's a fully functional Ubuntu install within a container on your host machine that took only a few seconds to setup!

When you're done, you can exit the container by entering `exit` at the command line. This will also stop the container.

Up next, we're going to explore [storage options](/intro-docker/docker-basics/docker-components/storage) for containers.
