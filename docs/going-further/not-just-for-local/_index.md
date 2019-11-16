---
title: "Other Docksal uses"
weight: 6
---

## Docksal can be used in scenarios other than local development

One common use for Docksal outside of local development is for running CI (continuous integration) tasks with services like TravisCI, CircleCI, or Bitbucket Pipelines.

### Why would you want to use Docksal for CI?

You might want to use Docksal for CI for the same reasons that you use Docksal for local development.

1. It reproduces many aspects of the production server
2. It's fast to spin up and tear down
3. It can be extremely verbose in output, helping you track down where issues are

### How to use Docksal for CI

Depending on which service you're using for your CI, the steps may be a little different, but the outcome is the same. We're not going to cover setting up CI with your project's repo in this training, but we will cover how to use Docksal in that service.

We're going to use [TravisCI](https://travis-ci.org) for our scenario here.

In TravisCI, we are given a blank slate to work with, meaning we can set it up however we want and run whatever commands are needed in order for the project to spin up and run tests. Travis, like many CI services, runs commands based off of a YAML file. We're only going to cover TravisCI from a very high level as it pertains to using Docksal for CI.

Let's take a look at a very basic `.travis.yml` file using Docksal and break it down:

```yaml
sudo: required
language: generic
dist: xenial

services:
  - docker

install:
  - bash <(curl -fsSL https://get.docksal.io)
  - fin version
  - fin sysinfo

before_script:
  - travis_retry fin init

script:
  - fin tests
```

Let's take a look at what this does:

``` yaml
sudo: required
language: generic
dist: xenial
```

1. TravisCI makes `sudo` available to our CI environment. This is needed for some of the commands the Dockerfile runs.
1. TravisCI is told to use the `generic` language which covers what we need to run the commands we're going to run.
1. TravisCI installs the `xenial` distribution of Ubuntu.

<!-- -->

``` yaml
services:
  - docker

install:
  - bash <(curl -fsSL https://get.docksal.io)
  - fin version
  - fin sysinfo
```

1. We tell TravisCI to install Docker as a required service.
2. We tell TravisCI to install Docksal.
3. Check that Docksal is installed by running `fin version` and `fin sysinfo`.

    If either of these commands fail, then we get a failed build and we'll have to look into it. If they succeed, then we can continue on.

<!-- -->

``` yml
before_script:
  - travis_retry fin init

script:
  - fin tests
```

1. `fin init` runs to spin up our project. If it fails during this step, our build fails and we need to find out why.
   **NOTE:** We prefixed this with `travis_retry`. This is a wrapper that will retry the following command 3 times before failing. Useful in the event that the healthcheck gets hung up or composer has issues.
2. The actual script to be run is `fin tests` which would be a custom command that defines tests to be run for a site. These can be PHPUnit, Behat, Cypress, Backstop, or any other type. Usually the command would be a wrapper for `fin exec phpunit...` or similar.

### Customizing for CI

If we wanted, we could create a custom command to run specifically for CI. This may be called `init-ci` and contain steps that are only important to testing or deploying within a CI environment.

We could also setup custom Docksal variables and services in `docksal-ci.env` and `docksal-ci.yml` by setting the `DOCKSAL_ENVIRONMENT=ci` environmental variable.

Let's say that even though you're using a host like Acquia and the Acquia stack on local, you don't need everything installed. You could probably do without Varnish, Memcache, and Solr. We could adjust these for the build by creating `docksal-ci.env` and `docksal-ci.yml`.

First, we would add

``` yml
env:
  global:
    - DOCKSAL_ENVIRONMENT=ci
```

to `.travis.yml`.

Then create our `docksal-ci.env` file

``` bash
$ cd .docksal
$ touch docksal-ci.env
```

Open this file and add

``` bash
DOCKSAL_STACK="default"
```

At this time, we don't need to change our PHP version or anything, so we won't create `docksal-ci.yml`.

### Summary

Docksal can be used for more than just local development. It can be used for CI and deployments and configured to run almost anything that you might need in a CI environment. Another use case is for setting up a sandbox environment on a server to test feature or show off projects. You can read more about that in the blog post [Installing Docksal on a remote server](https://blog.docksal.io/installing-docksal-on-a-remote-server-digital-ocean-scaleway-etc-ed0c230ddb82).



