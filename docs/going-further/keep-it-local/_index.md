---
title: "Using Local Files"
weight: 3
---

## Keeping it Local

### Introduction

There are a lot of times when certain information should not be put into version control. For example, API keys, sensitive user information, variables that change depending on who's using the repo, etc. For this, Docksal has local files available.

By default, Docksal operates in `local` mode. There is an environment variable set, `DOCKSAL_ENVIRONMENT=local`, that gives your projects access to some new files that you can keep outside of version control. Docksal will look for versions of `docksal.env` and `docksal.yml` that have the pattern `docksal-${DOCKSAL_ENVIRONMENT}` for their filenames and it will use them **in addition** to `docksal.env` and `docksal.yml`.

The loading order for the config files is:

1. `~/.docksal/docksal.env`
2. `~/.docksal/stacks/volumes-*.yml`
3. `~/.docksal/stacks/stack-*.yml`
4. `${PROJECT_ROOT}/.docksal/docksal.yml`
5. `${PROJECT_ROOT}/.docksal/docksal.env`
6. `${PROJECT_ROOT}/.docksal/docksal-${DOCKSAL_ENVIRONMENT}.yml`
7. `${PROJECT_ROOT}/.docksal/docksal-${DOCKSAL_ENVIRONMENT}.env`

As the next file loads, it overrides any matching settings in the previous file. So if your `~/.docksal/docksal.env` contained:

``` bash
FOO="foo"
BAR="bar"
BAZ="baz"
```

and your `${PROJECT_ROOT}/.docksal/docksal.env` contained:

``` bash
FOO="foo too"
```

your project would end up with:

``` bash
FOO="foo too"
BAR="bar"
BAZ="baz"
```

Remember, they only add or override settings. They will not remove them.

### A Local File Scenario - `docksal-local.env`

{{% notice info %}}
You are a member of a team using Docksal across multiple developers. The hosting provider requires that your requests to sync your database contain an API key in the request. Every member of the team has a different API key and for security, you do not want to let anyone's key get into the repo. This key will be used in a custom command to trigger a sync.
{{% /notice %}}

Going with what we've learned so far, this would probably make a great environment variable, so let's create our `docksal-local.env` file. We'll use our existing `docksal-training-projects` folder for this, so in a terminal or text editor get into the `.docksal` folder and create `docksal-local.env`.

``` bash
$ cd ~/projects/docksal-training-projects
$ cd .docksal
$ touch docksal-local.env
```

Now open this file and let's add in our API key and our associated user name. In your text editor open `docksal-local.env` and add the following.

``` bash
SECRET_API_KEY=12345678
SECRET_USERNAME=my-secure-name
```

Now save this file. We have our secure info, but we want to make sure this doesn't get into version control. Since we're using git for this training, we're going to put this file in our project's `.gitignore` file.

In your editor, open up `.gitignore` in your project's root folder and add the following line to the bottom.

``` bash
docksal-local.*
```

This will cover both `docksal-local.yml` and `docksal-local.env` and prevent them both from being tracked in version control.

We also need to run `fin project restart` for the changes to take effect. Any time we change an environment variable it's safer to run `fin project restart` instead of `fin up`.

### A Local File Scenario - `docksal-local.env`

{{% notice info %}}
You are a member of a team using Docksal across multiple developers. You've been tasked with prepping your project for an upgrade from an end of life version of PHP to a current version. Everyone else on your team is using PHP 5.6, but it's time to get ready for the current version. You need to be able to test code, even as it changes from other developers, to make sure it's ready for the upgrade.
{{% /notice %}}

In order to do this, we're going to use `docksal-local.yml` to override the image that we're using for our `cli` service, so let's create our `docksal-local.yml` file. We'll use our existing `docksal-training-projects` folder for this, so in a terminal or text editor get into the `.docksal` folder and create `docksal-local.yml`.

``` bash
$ cd ~/projects/docksal-training-projects
$ cd .docksal
$ touch docksal-local.yml
```

Open this file and override the image.

``` yaml
version: "2.1"
services:
  cli:
    image: "docksal/cli:2.9-php7.3"
```

This is all you need. Everything else will be kept from the other service definitions. You would then save this file and run `fin up` to get your version of the `cli` image with the current version of PHP. Since we added `docksal-local.*` to `.gitignore` this file won't be tracked and nobody else will be affected.

### Sharing Just Enough

It may not be apparent to everyone on your team that you need to create the `docksal-local.env` file for your project, and it can be a hassle to have to explain it again and again. One common practice is to keep a sanitized example file in the repo with instructions. Usually this is named something like `example.docksal-local.env` and looks similar to this:

``` bash
## Copy this file to docksal-local.env and update the values to your own
## username and API key

SECRET_API_KEY=
SECRET_USERNAME=
```

This file is not read by Docksal, does not keep any sensitive data, and can be kept in your version control without security concerns.

You could do the same with an `example.docksal-local.yml` file as well.

``` yaml
## To test the codebase against PHP 7.3, copy this file to docksal-local.yml and
## restart your project.
version: "2.1"
services:
  cli:
    image: "docksal/cli:2.9-php7.3"
```

### Summary

Not everything needs to be shared in the repo. Whether it's a custom override for your specific needs or sensitive information that shouldn't be in a public place, the local configuration overrides make it easier for you to keep the project shared, but keep your custom settings to yourself.

{{% notice tip %}}
The example files for this section can be found at https://github.com/JDDoesDev/docksal-training-projects/tree/local-files
{{% /notice %}}
