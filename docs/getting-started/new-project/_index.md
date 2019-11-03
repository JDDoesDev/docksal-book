---
title: "Beginning with a Boilerplate"
weight: 4
---

## Starting Easy

The easiest way to start a project with Docksal is to create a new one from a boilerplate. In this lesson we're going to use one of the existing starter projects to spin up a web application with just a few keystrokes.

### Do You Want to Build a Project?

We're going to begin by entering our terminal and going to a project folder. For demonstration purposes, I'm going to use `~/projects/`, but you're free to use whatever folder works for you. In upcoming lessons we will be pulling projects down from Github, but for this one we're going to use what's already in the system.

1. Open your terminal and go to your project folder.

    ``` bash
    $ cd ~/projects
    ```

1. Enter the command:
    ``` bash
    $ fin project create
    ```
    This will bring up a series of prompts to build our project.

2. At the first prompt, we'll name our project `my-first-docksal-application`
    ``` bash
    1. Name your project (lowercase alphanumeric, underscore, and hyphen): my-first-docksal-application
    ```

3. This will bring up a prompt to choose a type of project. We're going to start with a static HTML site.
    ``` bash
    1. What would you like to install?
      PHP based
        1.  Drupal 8
        2.  Drupal 8 (Composer Version)
        3.  Drupal 7
        4.  Wordpress
        5.  Magento
        6.  Laravel
        7.  Symfony Skeleton
        8.  Symfony WebApp
        9.  Grav CMS
        10. Backdrop CMS

      Go based
        11.  Hugo

      JS based
        12.  Gatsby JS
        13.  Angular

      HTML
        14.  Static HTML site

    Enter your choice (1-14): 14
    ```

4. Next, we'll be able to verify our setup with the following prompt:
    ``` bash
    Project folder:   /Users/my.username/projects/my-first-docksal-project
    Project software: Plain HTML
    Project URL:      http://my-first-docksal-project.docksal

    Do you wish to proceed? [y/n]: y
    ```

5. After confirming, our services will be created and our application will be running.

    **NOTE:** If you haven't run anything in Docksal on your system yet, you will see some images download. This is normal.
``` bash
Starting services...
Creating network "my-first-docksal-project_default" with the default driver
Creating volume "my-first-docksal-project_cli_home" with default driver
Creating volume "my-first-docksal-project_project_root" with local driver
Creating volume "my-first-docksal-project_db_data" with default driver
Creating my-first-docksal-project_cli_1 ... done
Creating my-first-docksal-project_web_1 ... done
Connected vhost-proxy to "my-first-docksal-project_default" network.
Waiting for project stack to become ready...
Project URL: http://my-first-docksal-project.docksal
Done! Visit http://my-first-docksal-project.docksal
```

1. Next, we're going to visit our newly created site by opening http://my-first-docksal-project.docksal in our browser.
    ![Static HTML site](/images/static.png)

    Not very much there, but as you can see, an entire web server is now running without having to configure any Apache, PHP, or any other part of a server stack.

### Let's examine our app:

```bash
my-first-docksal-project
├── .docksal
│   ├── docksal.env
│   └── docksal.yml
└── docroot
    └── index.html
```

As you can see, only three files on our system make up the basis for an entire web application, albeit a simple one. Behind the scenes, our Docksal system has created multiple new services, connected to existing ones, and mounted volumes for storage so that it can interact with the files on our host machine.

### Bringing the App Down

Because we may not want our app running for various reasons, we're going to stop it and remove it from the system.

1. Stop the services by running `fin project stop`
    ``` bash
    $ fin project stop
    ```

    This will stop the containers and services, but they will still be present on our system in an inactive state.

2. Now let's clean up by removing the services and volumes that Docksal created for this project by running `fin project remove`
    ``` bash
    $ fin project remove
    ALERT:  Removing containers and volumes of my-first-docksal-project
    Continue? [y/n]: y
    Removing containers...
    Stopping my-first-docksal-project_web_1 ... done
    Stopping my-first-docksal-project_cli_1 ... done
    Removing my-first-docksal-project_web_1 ... done
    Removing my-first-docksal-project_cli_1 ... done
    Removing network my-first-docksal-project_default
    Removing volume my-first-docksal-project_cli_home
    Removing volume my-first-docksal-project_project_root
    Removing volume my-first-docksal-project_db_data
    Volume docksal_ssh_agent is external, skipping
    ```

This stops and cleans out all of the services and volumes specific to this project. Look at the last line in the output, though.
``` bash
Volume docksal_ssh_agent is external, skipping
```

Why isn't this volume being stopped and cleared out? Well, it's for one of our shared services that is used by all Docksal projects on your system. If we had multiple projects and Docksal removed this while another project was running, then it could cause serious issues with your workflow.

### Summary

Creating an application with its own code, domain, and server in Docksal is simple. With the available boilerplates we have our choice of over a dozen starting points. In the next section we're going to spin up a fully functional Drupal site and look at some of the customizations Docksal does to run your applications.

