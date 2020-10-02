---
title: "Installation"
weight: 2
---

## Steps To Install Docksal

Now that we have everything in place to install Docksal, let's do just that.

{{% notice note %}}
Docksal's installer requires administrator access for some tasks to run. It adds commands to `/usr/local/bin`, edits a few secured files, and performs mounts, which may all require an elevated level of access.
{{% /notice %}}

### macOS with VirtualBox

This runs Docker inside a Virtual Machine with VirtualBox.

1. Install VirtualBox (a prerequisite)
2. If a notice appears to enable the Kernel extension, allow it

    `System Preferences > Security & Privacy`. If there is no notice, then it's already enabled.

3. Install Docksal by opening a terminal and running

    ``` bash
    $ bash <(curl -fsSL https://get.docksal.io)
    ```

### macOS with Docker for Mac

1. Install Docker for Mac (a prerequisite)
2. Start Docker for Mac and wait until the animation stops and/or the Docker menu says "Docker is running"
3. Install Docksal by opening a terminal and running

    ``` bash
    $ DOCKER_NATIVE=1 bash <(curl -fsSL https://get.docksal.io)
    ```

### Linux

Ubuntu, Mint, Debian, Fedora, CentOS, and derivatives are all supported. Check get.docker.com to see if you can install Docker.

1. Be sure that your system is prepared for Docksal and that you checked the [prerequisites](/installing-docksal/prereqs) before attempting to install.
3. Install Docksal by opening a terminal and running

    ``` bash
    $ DOCKER_NATIVE=1 bash <(curl -fsSL https://get.docksal.io)
    ```
