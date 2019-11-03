---
title: "Docker Storage"
weight: 5
---

## Docker Storage

As mentioned in our [containers](/intro-docker/docker-basics/docker-components/containers) section, a container is volatile. This means that if you stop a container, you lose everything that's not in the image the next time you start it back up. This can be painful if you're working on a containerized project that might take more than a day. Especially if your computer decides to update and reboot without warning.

Docker gets around this by using volumes and bind mounts. A volume is persistent storage for a Docker container, whereas a bind mount is "binding" a directory on your host machine to a container.

{{% children %}}
