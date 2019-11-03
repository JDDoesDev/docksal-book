---
title: "Beyond Docker"
weight: 3
---

## Why should we use more than just Docker?

### Considerations when using vanilla Docker

Docker is great in itself, but it can take some time to learn and to figure out the best ways to do things for your project(s). When beginning a new project that will use Docker there are several decisions to make:

* Will I need services or a single container?
* Will I need to build a custom Dockerfile?
* How will I connect my services?
* How will I interact with my application?
* What kind of storage will I use?
* What extras do I need to add into the stock container?
* Which images will work best for what I want to do?
* Does it make more sense to have a single image?
* How will I make changes if the requirements change?

A tool like Docksal and others take all of this into consideration and wrap it up into something that can be easily installed, easily configured, easily deployed, easily shared, and _most importantly_ easily used.

Let's look back on our [Docker Basics Section](/content/intro-docker/docker-basics/) and see what setting up a project can entail.

1. Find a base image
2. Add steps in a Dockerfile
3. Build the image
4. Decide if using a volume, bind mount, or something different
5. Spin up the container
6. Verify everything works
7. Ensure everything is setup so that we don't lose data if we stop the container

These steps are not difficult with some practice, but there are many tools that eliminate this setup, allowing you to begin development faster.

### Advantages to a tool that uses Docker

With a tool like Docksal, most of the setup is taken out of the equation and you can begin developing faster. Often times, most commands are wrapped up in a handy command so that long commands like `docker container exec my_container bash -lc "/var/www/vendor/bin/drush en stage_file_proxy"` can be handled in a more intuitive and less verbose manner.

In addition, tools like Docksal take all the guesswork out of deciding whether your application should exist in a single container or across multiple services that work together. They handle the orchestration between services out of the box so that you can get straight to developing.
