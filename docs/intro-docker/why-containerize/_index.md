---
title: "Why Containerize?"
weight: 2
---

## Containers vs Local Dev Environments vs Virtual Machines

For the purposes of this portion of the training, a "Local Dev Environment" refers to anything that lives on your computer's storage. Meaning MAMP, WAMP, local PHP, Apache, etc.

The key advantages to containerization are:

* **Reusability**
* **Portability**
* **Configurability**

### Reusability

* **Local dev environments** -- Require additional setup for each project. This means that a developer using one of the *AMP stacks or local services need to make sure that when they are working on a project, their settings match where the project will be hosted. Changing these settings can be time consuming and tedious.
* **Virtual Machines** -- Reusable, but they are far more resource intensive than containerization because they create an entire virtual machine, representing everything from the hardware, kernel, software, drivers, and every other aspect of a machine with each new virtual machine that is spun up. Multiple VMs on a single computer can take up a large chunk of resources.
* **Containerized dev environments** -- Can be reused on multiple systems, for multiple users, and for multiple projects. This means that no matter where a project is loaded, it will have the same setup as everywhere else from the initial install. Configuration time is minimal compared to other setups.

### Portability

* **Local dev environments** -- Usually not portable at all. They require setup on each machine where a project will live.
* **Virtual Machines** -- Portable, but different virtual machine setups can require that a developer download additional tools depending on which virtualized environment is being used in the project.
* **Containerized dev environments** -- Completely portable and only require that a developer have Docker installed on their system, eliminating the need to keep multiple VM tools up to date or multiple versions of the same service in your host machine's storage.

### Configurability

* **Local dev environments** -- Traditionally difficult to configure on a per-project basis.
* **Virtual Machines** -- Highly configurable, but time consuming to remove and recreate a VM with changed settings.
* **Containerized dev environments** -- Highly configurable and fast to update, often only taking seconds to recreate an environment, even after significant changes since only the affected containers need to be updated and not the entire project.

