---
title: "Docksal Stacks"
weight: 3
---

## Stacks in Docksal

### What is a stack?

In Docksal, the concept of a "stack" is a collection of configurations or services put together to create a reusable application. They are built in `yml` files, and stored within the Docksal folder at `~/.docksal/stacks/`. A stack `yml` file is the equivalent to a `docker-compose.yml` file in that it defines services to be used for an application.

Let's examine the Docksal `default` stack:

``` yaml

# Basic LAMP stack
# - Apache 2.4
# - MySQL 5.6
# - PHP 7.2

version: "2.1"

services:
  # http(s)://VIRTUAL_HOST
  web:
    extends:
      file: ${HOME}/.docksal/stacks/services.yml
      service: apache
    depends_on:
      - cli

  db:
    extends:
      file: ${HOME}/.docksal/stacks/services.yml
      service: mysql

  cli:
    extends:
      file: ${HOME}/.docksal/stacks/services.yml
      service: cli
```

You can see that there are three services defined as part of the default stack. When we use this stack for our application, we get a `web` service, a `db` service, and a `cli` service. As we learned about in the [services](/intro-docker/docker-basics/docker-components/services/) section, these services are images that are built together as part of a larger application.

One thing to notice is that there isn't a lot going on here. In fact, it seems oddly simple for something so powerful. Let's dive into how Docker and Docksal take this file and convert it into a full application.

First we're going to look at what comes before the services

``` yaml
version: "2.1"

services:
```

The important line here is `version: "2.1"`. This line tells Docker which Docker Compose API we're running against so that even if Docker is upgraded, our application will not break. Take note of the version here when defining your own services, as some commands may not be available in the version we're using for Docksal.

Now let's examine the `web` service in the default stack.

First off, the `web` service is where our webserver lives. In the default stack, we're using Apache. All web requests are sent through this service and all rendering happens here as well.

Now the stack:

``` yaml
services:
  # http(s)://VIRTUAL_HOST
  web:
    extends:
      file: ${HOME}/.docksal/stacks/services.yml
      service: apache
    depends_on:
      - cli
```

Let's examine this in chunks:

``` yaml
  web:
```

Here we're naming our service. This could literally be anything, but it makes sense to name it descriptively.

``` yaml
    extends:
      file: ${HOME}/.docksal/stacks/services.yml
      service: apache
```

In this section we're telling Docker Compose where to find the actual configuration. Docksal places all of the common services inside a file at `${HOME}/.docksal/stacks/services.yml`. Each of those services are named and available to be used by other files. As you can see here, we're taking the service `apache` from `${HOME}/.docksal/stacks/services.yml` and loading it into our application.

``` yaml
    depends_on:
      - cli
```

Finally, we're making sure that the `cli` service is started before this one. This is because the our web service needs the `cli` service to be running so that it can function properly.

Now let's look at what's in the `apache` service in `${HOME}/.docksal/stacks/services.yml`

``` yaml
services:
  # Web: Apache
  apache:
    hostname: web
    image: ${WEB_IMAGE:-docksal/apache:2.4-2.3}
    volumes:
      - project_root:/var/www:ro,nocopy  # Project root volume (read-only)
    labels:
      - io.docksal.virtual-host=${VIRTUAL_HOST},*.${VIRTUAL_HOST},${VIRTUAL_HOST}.*
      - io.docksal.cert-name=${VIRTUAL_HOST_CERT_NAME:-none}
      - io.docksal.project-root=${PROJECT_ROOT}
      - io.docksal.permanent=${SANDBOX_PERMANENT:-false}
    environment:
      - APACHE_DOCUMENTROOT=/var/www/${DOCROOT:-docroot}
      - APACHE_FCGI_HOST_PORT=cli:9000
      - APACHE_BASIC_AUTH_USER
      - APACHE_BASIC_AUTH_PASS
    dns:
      - ${DOCKSAL_DNS1}
      - ${DOCKSAL_DNS2}
```

There's a lot more going on here than in `default-stack.yml`, right? Let's break this service down a bit to explore what's going on.

``` yaml
  apache:
    hostname: web
    image: ${WEB_IMAGE:-docksal/apache:2.4-2.3}
```

We are defining a hostname for our service and calling it `web`. Again, the name is arbitrary for the file, but it is important for the way Docksal works. We're also defining which image our service is going to be built on.

Take notice of the bash scripting that is being used here: `${WEB_IMAGE:-docksal/apache:2.4-2.3}`

This is looking for an environment variable by the name of `WEB_IMAGE`, and if it doesn't exist, then load the default. We'll talk more about configuring environment variables in an upcoming section.

This line is the equivalent of running a `docker run` command with arguments and flags. Let's build that command: `docker run --name apache --hostname web docksal/apache:2.4-2.3`. If you were to run that command, you would get a simple Apache container, but it wouldn't be a part of our application.

``` yaml
    volumes:
      - project_root:/var/www:ro,nocopy  # Project root volume (read-only)
```

Even though it says volumes, don't be fooled. This is creating a bind mount from the `project_root` volume to the folder `/var/www` within the container. We're also telling Docker that it's read-only (`ro`) and we don't need to copy anything to the container, just create pointers to the files there (`nocopy`).

We're using something called "Named Volumes" here, which makes it easier to configure the bind mount as well as inspect the bind mount from the command line.

``` yaml
    labels:
      - io.docksal.virtual-host=${VIRTUAL_HOST},*.${VIRTUAL_HOST},${VIRTUAL_HOST}.*
      - io.docksal.cert-name=${VIRTUAL_HOST_CERT_NAME:-none}
      - io.docksal.project-root=${PROJECT_ROOT}
      - io.docksal.permanent=${SANDBOX_PERMANENT:-false}
```

Labels are specific to Compose projects. These are key value pairs that are used for orchestration of the application's services. For example, the line `- io.docksal.virtual-host=${VIRTUAL_HOST},*.${VIRTUAL_HOST},${VIRTUAL_HOST}.*` is used to create a label that Docksal uses to tell the web service which domains it should be listening for while it's running.

``` yaml
    environment:
      - APACHE_DOCUMENTROOT=/var/www/${DOCROOT:-docroot}
      - APACHE_FCGI_HOST_PORT=cli:9000
      - APACHE_BASIC_AUTH_USER
      - APACHE_BASIC_AUTH_PASS
```

The environment section passes environment variables into the service. These can be set elsewhere, as part of the host machine's environment, or in this section. As you can see, the `APACHE_DOCUMENTROOT` variable is being set in the service definition, whereas `APACHE_BASIC_AUTH_USER` is being passed from host to service.

``` yaml
    dns:
      - ${DOCKSAL_DNS1}
      - ${DOCKSAL_DNS2}
```

Finally, the `dns` section is telling the web service what DNS service to use. These variables are set at runtime.

### Summary

Docksal stacks make it extremely easy to get working with minimal, even _zero_ configuration. With the default stacks that are packaged with Docksal, we can have a robust development environment running without having to worry about setting up PHP, Apache, MySQL, or others. Simply start Docksal in your project and begin developing.

Up next, we're going to learn about the services that make Docksal work. [Continue on >](placeholder)
