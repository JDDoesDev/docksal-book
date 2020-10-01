---
title: "Docksal Services"
weight: 2
---

## Project and System Services

Docksal services are divided into two groups: **Project** and **System**.

**Project Services** are services that are defined and run at the project level. This means that you can have multiple instances of a web or cli service running at the same time on for different projects. For example, if we were working on a Drupal site for a client who uses Pantheon for hosting, we would want our web service to run Nginx. However, a ticket comes in from another client on Acquia that needs to be handled in a hurry.

We could stop the current project, but if we forget then we're not stuck with Ngnix. We can spin up the Acquia project's Docksal environment and get the proper version of PHP along with Apache that only touches that project's codebase.

**System Services** are services that are shared between all projects. These include the SSH agent, the DNS service, and the Reverse Proxy service. These are all available to every project and every project uses them.

Let's explore these services.

### Project Services

Project services can be configured on a per-project basis. The three main services are `web`, `db`, and `cli`. However, Docksal ships with a few extras that can be configured to be used with your project. We will explore these extras more in the [Docksal Files](placeholder) section.

The default stack contains these project services:

* `cli`

    The default `cli` service runs `supervisord`. This is a client/server system that Docksal uses to control processes on the service. For `cli` these processes are the daemons `php-fpm`, which we use for running our PHP scripts, `crond`, which executes scheduled commands, and `sshd`, which is used to allow SSH connections.

    It uses the `docksal/cli:2.11-php7.3` image out of the box which contains PHP 7.3, Composer, Drush, Drupal Console, WP-CLI, Terminus, and Platform along with many more. It also has Nodejs v12.18.1, Ruby 2.7.1, Python 3.8.3, and msmtp. These inclusions are useful for having some of the most commonly used languages and compilers/interpreters used in web application development.

* `db`

    The default `db` service runs the `mysqld` daemon. This allows connections to the database layer and separates the database layer from the CLI layer to simulate a real hosting environment.

    It uses the `docksal/mysql:5.6-1.5` image out of the box with MySQL 5.6.

* `web`

    The default `web` service runs Apache server 2.4. This is responsible for maintaining a web connection and allowing you to connect your browser to your application.

    It uses the `docksal/apache:2.4-2.3` image out of the box.

### System services

System services are shared across all Docksal projects. These are reused for a lot of the functionality that makes Docksal works without having to maintain multiple installs of the same software.

The default system services:

* `docksal-dns`

    This is the service that is responsible for allowing your projects to use custom domain names. It defaults to using the domain `project-name.docksal` where `project-name` is the root folder of your project. So if your project lived at `/Users/me/projects/cats` the domain you would access is `cats.docksal`. Me-wow!

    It does rely on an internet connection, but we'll cover how to get around that a bit later.

* `docksal-ssh-agent`

    This service allows you to use your host machine's SSH keys within your projects. This is handy for pulling databases from hosting providers or using `git` commands within the container. Without this, you would have to add a key to the container every time you spun it up and make sure that it matches the keys on whatever external services you use that require SSH authentication.

* `docksal-vhost-proxy`

    This service allows multiple domain names to be routed to the appropriate containers. This allows us to have several projects running, each with a custom domain, and have them routed to the correct web service based on the host name.

In addition, networking, shared volumes, and other functionality we will cover later are part of the system services.

### Summary

Project and system services allow our projects to remain separate from each other, but to share quite a bit of "under the hood" functionality across the board. Project services are spun up on a per project level, whereas system services are running at all times and used by all projects.

Next, we'll explore some of the starter-kits that are available with Docksal.
