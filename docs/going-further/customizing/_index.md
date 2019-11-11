---
title: "Customizing a Project"
weight: 1
---

## Make Our Drupal Project Our Own

In the last section, we installed a Drupal site using a boilerplate Composer-based template, but what if we want to make changes to some of our installation settings? In this section we're going to look at a few places where we can customize our project to make it a little more our own.

Let's start off by making some adjustments within our `docksal.env` file.

### Customizing `docksal.env`

The `docksal.env` file contains environmental variables for Docksal, outside of the project. These are variables that the project looks for when starting up in order to run the correct settings, build the right services, and others. We're going to start by changing our stack to match a production environment.

In your favorite text editor or IDE of choice, open `~/projects/docksal-training-projects/.docksal/docksal.env`

The default settings here are:

``` shell
DOCKSAL_STACK=default
DOCROOT=web
MYSQL_PORT_MAPPING='0:3306'
XDEBUG_ENABLED=0
COMPOSER_MEMORY_LIMIT=-1
```

**NOTE:** Comments left out for brevity.

We're going to make a couple of changes now. Let's say we're using Pantheon for hosting. In order to mimic the Pantheon production environment, we can change our stack. Change `DOCKSAL_STACK=default` to `DOCKSAL_STACK=pantheon` and save the file.

This won't do anything until we update our project. We're going to do that using the command `fin up`.

In your terminal, run

``` shell
$ fin up
```

This will trigger all of your services to restart and update any that have changed.

If we look at the Pantheon stack we'll see that it's different from the default stack in a number of ways. This file can be found at `~/.docksal/stacks/stack-pantheon.yml`

The Pantheon stack includes:

* Nginx 1.14
* MariaDB 10.1
* PHP 7.2
* Varnish 4.1
* Redis 4.0
* ApacheSolr 3.6

And these changes will be reflected when you run `fin up`.  Let's look at the output:

``` shell
docksal-training_cli_1 is up-to-date
Recreating docksal-training_db_1    ... done
Recreating docksal-training_web_1 ... done
Creating docksal-training_solr_1  ... done
Creating docksal-training_redis_1 ... done
Creating docksal-training_varnish_1 ... done
Waiting for project stack to become ready...
Waiting for project stack to become ready...
Project URL: http://docksal-training.docksal
```

As you can see, we've created a `solr`, `redis`, and `varnish` service in our project now. This gives us access to these services locally to mimic settings we may have on our production server.

Great, now let's make some more edits.

We're going to enable Xdebug and change our project's domain through `docksal.env`. Change the line `XDEBUG_ENABLED=0` to `XDEBUG_ENABLED=1` and add the line `VIRTUAL_HOST=mycustomsite.docksal` in your `docksal.env` file.

Save and run `fin up` again.

Let's look at the output:

``` shell
$ fin up
Starting services...
Recreating docksal-training_cli_1 ...
Recreating docksal-training_solr_1 ...
Recreating docksal-training_cli_1  ... done
Recreating docksal-training_solr_1    ... done
Recreating docksal-training_web_1  ... done
Recreating docksal-training_varnish_1 ... done
Waiting for project stack to become ready...
Project URL: http://mycustomsite.docksal
```

Notice how our `Project URL:` has changed to reflect our new domain. Go ahead and visit it to make sure everything is still working.

### Customizing

As a hypothetical, let's say that we want to change the site name on install. We could do this by simply editing `.docksal/commands/init-site` which makes sense, but alternatively we could add an environment variable to `docksal.env` and alter `init-site` once. Let's try it.

1. In your text editor, open `docksal.env`
2. Add the line `SITE_NAME="This Site Has A Different Name"` to the bottom of the file.
3. In your text editor, open `.docksal/commands/init-site`
4. Look for the lines that install Drupal using Drush, around line 89.
5. Change the line that starts with `--site-name=` to `--site-name="${SITE_NAME}"`
    * **NOTE:** The double-quotes are important here.
6. Save both files.

If we were to run `fin init` now, the site name wouldn't work. That's because, as mentioned before, the `init-site` script runs _inside_ the `cli` container. We need to pass this variable so that the `cli` container has access.

To do that, we're going to need to alter the `docksal.yml` file, but first, what is `docksal.yml`?

`docksal.yml` is a `docker-compose` file that defines and helps tie services together. Our default `docksal.yml` is pretty bare, but it contains something that will help us out. Let's take a look. Open `.docksal/docksal.yml`. You should see:

``` yml
version: "2.1"
services:
  cli:
    environment:
      - COMPOSER_MEMORY_LIMIT
```

This doesn't define any services by itself. Those are defined in the Docksal stacks, but in the processing order, Docksal looks for this file and uses it to alter the default behavior or the current `cli` service. Right now we're passing in the `COMPOSER_MEMORY_LIMIT` variable that we have defined in `docksal.env`.

To get access to our new site name, we need to tell Docksal to pass the `SITE_NAME` variable as well.

1. In your text editor open `docksal.yml`
2. In the `environment` section, add the line `- SITE_NAME`
    **NOTE:** Indentation of this line must match the indentation of the `- COMPOSER_MEMORY_LIMIT` line, otherwise the YAML will not work.
3. Save your file

Now we're ready to run `fin init`. After everything has run, visit your site at `http://mycustomsite.docksal/` and notice that the site name has changed. Now we can easily change the site name in one spot.

### Going Further with Customization

There are many other customizations that we can do with our `docksal.env` and `docksal.yml` files. We can add labels to services which are used to define functionality, we can change domain names, and we can even define our own services and variables to be used for our projects. In the next section we're going to explore some more advanced customizations using the `docksal.env` and `docksal.yml` files.

{{% notice info %}}
**NOTE:** The code for this section can be found in the `drupal-site-step-2` branch of [github.com/JDDoesDev/docksal-training-projects].
{{% /notice %}}
