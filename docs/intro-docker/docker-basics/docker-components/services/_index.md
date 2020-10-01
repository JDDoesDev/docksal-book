---
title: "Docker Services"
weight: 6
---

## Docker Services

Sometimes it makes sense to wrap up a few containers to work together. For example, if we don't want to put too much on a single container to handle a database, a web server, networking, etc. we can create services in place of standard containers. Docksal relies on services extensively to create various stacks.

Services are often part of a larger application whereas containers usually stand on their own. One of the main advantages of services compared to containers is that the pieces can easily be swapped out. Let's say that we wanted to move hosting providers for our web application. Provider A uses Apache and PHP 7.1 and provider B uses Nginx and PHP 7.3. We could just push everything up to provider B and _hope_ it works, but with services we can create a custom local environment that matches everything on provider B and make the changes locally. This way we can feel confident when we deploy, knowing that our application works well inside the hosting environment.

Many of the functions of services are beyond the scope of this section of the training, however a few things that should be noted are:

* Docker services are defined individually as part of a larger application
* Docker services are structured in Dockerfiles, however they are managed by `docker-compose`, a utility that interacts with an entire application rather than a single container
* Applications that use services are defined in a `*.yml` file, often named `docker-compose.yml` with several services defined along with settings and information to help everything run together

The [Docker docs](https://docs.docker.com) have many examples of using services and it's something that we will cover more in the Configuring Docksal section later.
