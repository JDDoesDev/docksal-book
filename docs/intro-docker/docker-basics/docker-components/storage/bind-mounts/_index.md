---
title: "Bind Mounts"
weight: 2
---

## Bind Mounts

A bind mount is very similar to a volume, in fact it uses a lot of the same concepts. However, instead of creating a volume with nothing in it, a bind mount _binds_ to a folder on your host machine and _mounts_ it within your container. As we'll find out later, Docksal relies heavily on this concept for local development, but we'll get into that in a bit.

When we set up a bind mount we are doing two things. We're telling our host machine to export the source folder to a container, and we're telling a container to create a matching folder on the container to place that source folder. Essentially we're linking the data from the source, putting in the container where it has the same functionality of a volume, and we'll have access to changes in the data on the host machine within our container.

This is useful for shared projects using version control systems like git. In fact, many projects are using this type of workflow to duplicate development environments across many developers so that there are no surprises when deploying to a hosting provider. This workflow usually consists of having a codebase including some sort of Dockerfile or image included. A developer pulls down the codebase and runs the commands to start and run a container. Once the container is running and connected to the bind mount, any changes on the host machine's files will be reflected immediately in the container.

A bind mount itself is not a volume, but a volume can use a bind mount. Let's try an example.

First, from our host machine, let's create a volume:

``` bash
$ docker volume create bindMountTest
```

Now let's take a look to see if our volume exists:

``` bash
$ docker volume ls
DRIVER              VOLUME NAME
local               bindMountTest
```

Perfect! Now, let's spin up a container using that volume:

``` bash
$ docker run -it \
  --name bindtest \
  --mount source=bindMountTest,target=/app \
  ubuntu /bin/bash
root@3ffc3d80141d:/#
```

For a breakdown of this command, please revisit the [volumes section](/intro-docker/docker-basics/docker-components/volumes).

Now that we're in our container we can run `ls -al` and see a printout of all of our folders within our `/` or root folder.

``` bash
root@3ffc3d80141d:/# ls -al
total 76
drwxr-xr-x   1 root root 4096 Oct 26 18:13 .
drwxr-xr-x   1 root root 4096 Oct 26 18:13 ..
-rwxr-xr-x   1 root root    0 Oct 26 18:13 .dockerenv
drwxr-xr-x   2 root root 4096 Oct 26 18:09 app <--- This is where our bindMountTest volume is mounted.
drwxr-xr-x   2 root root 4096 Oct 10 21:07 bin
drwxr-xr-x   2 root root 4096 Apr 24  2018 boot
drwxr-xr-x   5 root root  360 Oct 26 18:13 dev
[The rest of the output is omitted for brevity.]
```

If we `cd app` we'll see that there's nothing inside this folder, which is fine. We don't have anything in our volume that we're expecting to show up.

Let's exit our container and remove it.

``` bash
root@3ffc3d80141d:/# exit
$ docker container stop bindtest
$ docker container rm bindtest
```

Now, let's create some data to test out a bind mount.

First, we're going to need a folder to use as our host data so let's create a `~/docksal-training/` folder. From your home folder, run `mkdir docksal-training` and `cd docksal-training`.

Now let's create a file.
``` bash
$ touch bind-mount-test
$ echo "Testing a bind mount" >> bind-mount-test
$ cat bind-mount-test
```

You should see the output `Testing a bind mount`.

Since Docker runs system-wide we can run the `docker` command anywhere, so we'll stay in our `docksal-training` folder for now. Next, we're going to spin up a container using a bind mount. It is much the same as using a volume with two differences:

1. You'll notice that we're using a `type=bind` argument for the command.
2. When spinning up a container, Docker will automatically create a volume if it does not exist. This is not the case if we use a folder for `source` and the `source` folder does not exist. We will see an error and the build will fail.

Okay, let's spin up a container with a bind mount and see what happens.

``` bash
$ docker run -it \
  --name bindtest \
  --mount type=bind,source="$(pwd)",target=/app \
  ubuntu /bin/bash
```

{{% notice info %}}
**NOTE:** The `$(pwd)` in the above command references the current directory where the command is run.
{{% /notice %}}

Now, if we go to the `/app` folder of our container, we should see the file `bind-mount-test`

``` bash
root@04899aa02741:/app# ls -al
total 8
drwxr-xr-x 4 root root  128 Oct 26 20:02 ./
drwxr-xr-x 1 root root 4096 Oct 26 20:00 ../
-rw-r--r-- 1 root root   21 Oct 26 20:02 bind-mount-test
```

Let's see what's in the `bind-mount-test` file.

``` bash
root@04899aa02741:/app# cat bind-mount-test
Testing a bind mount
```

Look at that, we've brought our file from the host into the container! Wonderful! Now let's do something with it.

In a new terminal window on your host machine, let's do `echo "And now I'm changed from the host" >> bind-mount-test` within our `docksal-training` folder.

``` bash
$ cat bind-mount-test
Testing a bind mount
And now I'm changed from the host
```

Let's go back to our container terminal and see what happened.
``` bash
root@04899aa02741:/app# cat bind-mount-test
Testing a bind mount
And now I'm changed from the host
```

Outstanding! We now have the ability to change files in the container from our host which means that we can develop on our host and use the power of containers to build and host our code in a repeatable manner.

There are a lot more feature of bind mounts and other mounts that are beyond the scope of this training. For further reading please check out the [Docker Docks](https://docs.docker.com/storage/bind-mounts/).

Up next, we're going to take a look at Docker registries.
