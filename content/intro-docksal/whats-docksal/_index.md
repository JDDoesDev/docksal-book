---
title: "What's a Docksal"
weight: 1
---

## What is Docksal?

Docksal is a containerized development tool that is setup for web application development. Docksal uses Docker and Docker compose to create reusable application development environments that can be shared across all developers, as well as for testing, and even production environments.

### A Brief History of Docksal

Docksal began its life as an open sourced development management tool created by [FFW](https://ffwagency.com) known as **DRUDE** for **Dr**upal **D**evelopment **E**nvironment. The description from the [Drude introduction blog post](https://ffwagency.com/learning/blog/simplify-local-development-drude) says that it "brings together common development tools, minimizes configuration, and ensures environment consistancy everywhere in your continuous integration workflow."

Initially it was an internal tool, but being the open source advocates that they are, FFW released it into the wild for anyone to use and benefit from.

After a bit of time, it was expanded to include more than just Drupal and the name Drude was no long as valid as it once was. This expansion included out of the box support for "WordPress, stand-alone HTML files, or any other PHP project." ([FFW Announcing Docksal blog post](https://ffwagency.com/learning/blog/announcing-docksal-docker-based-development-environment))

Over time it's grown into a powerful, robust, and highly configurable development tool used by developers around the world to speed up their development time and help eliminate "it worked on my local!" issues. At the time of this writing there are 46 contributors to the core Docksal project, and several others who contribute to images, addons, and other aspects of the project.

### Where Docksal is Now

Docksal is currently used by several agencies and development shops as a tool to speed up development, test code rapidly, and deploy with ease. It's become one of the top developer tools, including out of the box support for:

* Drupal 8
* Drupal 7
* Wordpress
* Magento
* Laravel
* Symfony Skeleton
* Symfony WebApp
* Grav CMS
* Backdrop CMS
* Hugo
* Gatsby JS
* Angular

With more being considered all the time.

### How is Docksal different than X?

I would like to preface this with the statement that I have been using only Docksal for the past year, so instead of a comparison between Docksal and other environments, I'm going to list some things that separate it from others.

* **Switching between Docker modes**

    Docksal allows you to easily change between Docker on a Virtual Machine and Docker for Mac /Docker for Windows. This is not a consideration on Linux. Before you call out that we compared containerization and virtualization a few sections ago, please remember this: Docker for Mac / Docker for Windows (let's call it Docker Native from now on) is itself a virtual machine. Docker requires the Linux kernel to function, so any OS that isn't Linux-based is going to require some sort of virtualization.

    With a simple command you can change from using a VM like VirtualBox to Docker Native or vice versa. This is extremely beneficial because as of the time of this writing, Docker Native can be very, very slow compared to using a VM.

* **Docksal is written in Bash**

    This means that on Mac and Linux, no additional software is needed to run Docksal, where others may require Ruby, Nodejs, Python, or others. Because of this, setup time is decreased. On Windows, WSL (Windows Subsystem for Linux) is required, meaning that it does not work older versions of Windows and requires WSL be enabled and Ubuntu be installed to run commands.

    With the Windows exception, using Bash instead of another language keeps in the spirit of containerization by reducing requirements of the host machine so that development tools are separated from the host machine.

* **Docksal is easily configurable**

    Docksal exposes many settings so that users can extend existing services or easily create their own based on a number of languages and setups. Within the [Docksal repo](http://github.org/docksal) there are several ready-to-install images that are available on [Docker Hub](https://hub.docker.com) for multiple setups.

    In addition, creating your own service is relatively straightforward and lets you tie in to the greater application without having to have an extensive knowledge of Docker services and orchestration.

* **Out of the Box / Zero Configuration Setup**

    Spinning up a container for your project is as simple as installing Docksal with a Bash script, and adding a single folder.

* **Configuration for Hosting Providers**

    There are stacks in place to mimic Pantheon and Acquia two of the more popular Drupal hosting providers. In addition, there are built in commands to use Pantheon's `terminus` and Platform.sh's `platform` CLI utilities by default, allowing for easier local development closely coupled with your hosting provider.

Up next, we're going to take a look at some of the default stacks available out of the box with Docksal.  [Continue on >](/intro-docksal/stacks)
