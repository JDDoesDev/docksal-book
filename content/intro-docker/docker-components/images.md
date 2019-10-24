---
title: "Docker Images"
weight: 3
---

### Images

Images are blueprints for containers. An image itself is often defined by no more than a single file, usually some form of a Dockerfile. This file gives the image instructions on what it needs to be run. Take the following, very simple example:

``` dockerfile
FROM docksal/cli:2.6-php7.3

RUN echo "Hello, world!"
```

This file is using another image `docksal/cli:2.6-php7.3` and building off of that to create its own image. The only thing this image will do is create a container using Docksal's CLI image and then echo the words `Hello, world!` in the build output. Nothing too fancy, but there is a lot going on here.

Let's build this simple image and see what happens:

``` shell
❯ docker build --tag "test:file" /Users/jdflynn/example/
Sending build context to Docker daemon  50.44MB
Step 1/2 : FROM docksal/cli:2.6-php7.3
 ---> bb4c72e4b656
Step 2/2 : RUN echo "Hello, world!"
 ---> Using cache
 ---> d2c00859cae2
Successfully built d2c00859cae2
Successfully tagged test:file
```
Doesn't seem like a lot, but if this were your first time building this image, you would see a lot of status indicators that look like this:

``` shell
177e7ef0df69: Already exists
9bf89f2eda24: Already exists
350207dcf1b7: Already exists
a8a33d96b4e7: Already exists
82350ee8f11f: Pulling fs layer
2d9047762251: Pulling fs layer
196d943fac59: Pulling fs layer
ff00d78cbcf3: Pulling fs layer
8b971b61b7b6: Pulling fs layer
337d6d904976: Downloading [==============================>                    ]  7.646MB/12.49MB
20c027cb1a77: Waiting
ba27c2e2de1c: Waiting
```
What this is doing is following the instructions provided by the Dockerfile to build an image. From our two-line Dockerfile, we're pulling instructions in from another Dockerfile and from another until we reach the base file, most likely ending up at `scratch`, a base image provided on Docker Hub.

So now we have an image, but it's not doing us a lot of good. So, what's next?

[Docker Containers](/intro-docker/docker-components/containers/)

