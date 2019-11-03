---
title: "Volumes"
weight: 1
---

## Volumes

The advantage to a volume is that it can be easily wrapped up with a container and be portable along with an image, meaning that one could wrap up everything together, the image and the volume, and ship or share it.

To create a volume, we need to define it. We do that by either creating it on its own, starting a container with an attached volume, or starting a service with an attached volume. The easiest way is to create it.

`$ docker volume create example`

Now if we look at what volumes are available, using `docker volume ls` we'll see the volume `example` listed, however this doesn't do a lot other than reserve some storage on your host for the volume. What we really want to do is have a volume mounted to a container. We do this using the `docker run --mount` command.

{{% notice info %}}
**NOTE:** The `--mount` flag and the `-v` flags will achieve the same things, but with different syntax. The `--mount` flag is easier to use and has a clearer syntax, so that's what we'll be using for these examples.
{{% /notice %}}

To start a container with a volume, we'll use the following command:

``` bash
$ docker run -i -t \
  --name exampleContainer \
  --mount source=exampleVolume,target=/app \
  ubuntu /bin/bash
```

Let's go over this line by line:
```
docker run -i -t \
```

This tells docker that we're going to run a container, we want it to be interactive `-i`, and we want to be able to use the terminal to interact with it `-t`.

```
  --name exampleContainer \
```

We're naming our container `exampleContainer`

```
  --mount source=exampleVolume,target=/app \
```

We're using the volume `exampleVolume` for this container and mounting it to the `/app` folder within the container.

```
  ubuntu /bin/bash
```

Finally, we're using the default `ubuntu` image and using the Bash shell for interacting with it.

If this works, then we should see something that looks like this:

``` bash
root@8bf1f5964b3d:/#
```

Where the string after the `root@` will vary based on your hash.

Now let's try an experiment, shall we? While we're inside this container we just created, we're going to create two files. One in the `/app` folder, and one in the user folder, in this case `/root`.

Run the following commands:

``` bash
$ echo 'Test volume file' >> /app/test

$ echo 'Test container file' >> ~/container-test
```

We can check that these files exist by running `cat /app/test` and `cat ~/container-test` which should display the expected text.

> NOTE: the `~` in bash is a shortcut to the current user's home folder. For root, this is `/root`, but for other users it is usually `/Users/<my user name>`.

Now, let's exit our container by typing `exit`. Since we don't have anything running in the container this will cause it to exit.

Run `docker container rm exampleContainer` to remove the container.

Finally, recreate the container using the command from above:

``` bash
$ docker run -i -t \
  --name exampleContainer \
  --mount source=exampleVolume,target=/app \
  ubuntu /bin/bash
```

This will get us back into our `exampleContainer`. Notice that we have a new hash in the command line prompt.


Go to your `/root` folder using `cd /root` or `cd ~` and run `ls -al` to see what files are there. You should see output similar to

``` bash
root@af040d25a903:~# ls -al
total 16
drwx------ 2 root root 4096 Oct 10 21:07 .
drwxr-xr-x 1 root root 4096 Oct 26 15:43 ..
-rw-r--r-- 1 root root 3106 Apr  9  2018 .bashrc
-rw-r--r-- 1 root root  148 Aug 17  2015 .profile
```

Our test container file is gone forever. This happened because we removed the container and everything in it disappeared.

Now, go to your `/app` folder using `cd /app` and run `ls -al` to see what files are there. You should see output similar to

``` bash
root@af040d25a903:/app# ls -al
total 12
drwxr-xr-x 2 root root 4096 Oct 26 15:32 ./
drwxr-xr-x 1 root root 4096 Oct 26 15:43 ../
-rw-r--r-- 1 root root   17 Oct 26 15:32 test
```

Run `cat test` and you'll see our `Test volume file` text from before. Why is this? It's because we mounted a volume to the container in the `/app` folder. The data from the volume is accessible to the container, but it isn't tied to the container. In fact, it lives on our host machine! We could visit it without even spinning up a container if we wanted to.

Let's exit this container and remove it. Run `exit` from the command line of the container and once in your host terminal run `docker container rm exampleContainer`. We'll also remove the example volume by using `docker container rm exampleVolume`.

Next, we'll learn about the other primary method of persistent storage in Docker: Bind mounts.

For further reading on volumes, check out https://docs.docker.com/storage/volumes/
