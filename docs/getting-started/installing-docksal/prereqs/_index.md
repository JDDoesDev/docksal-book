---
title: "Before Installing"
weight: 1
---

## Prereqs and System Requirements

The prerequisites for installing Docksal are pretty slim, but important.

* ### All systems
    * 8GB RAM or more
    * Command Line access (Terminal, iTerm, or an IDE terminal)

* ### Mac
    * 2010 or newer model
    * Docker for Mac or VirtualBox (**NOTE:** This training is currently based on using VirtualBox)

* ### Linux
  * By default, Apache listens on `0.0.0.0:80` and `0.0.0.0:443`. This will prevent Docksal reverse proxy from running properly. You can resolve it an any of the following ways:
      * Reconfigure Apache to listen on different host (e.g., 1`27.0.0.1:80` and `127.0.0.1:443`)
      * Reconfigure Apache to listen on different ports (e.g., `8080` and `4433`)
      * Stop and disable Apache


  * Check that you have installed and configured:

      * `curl`
      * `sudo`
      * CPU with SSE4.2 instruction set supported
      * Docker

* ### Windows
    * Windows 10 with Windows Subsystem for Linux and CPU with hardware virtualization (VT-x/AMD-V) supported and enabled in BIOS.
    * Ubuntu 18.04 or greater
    * Docker for Windows or VirtualBox (**NOTE:** This training is currently based on using VirtualBox)
