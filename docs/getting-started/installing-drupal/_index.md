---
title: "Installing a Drupal Site"
weight: 5
---

## Let's Do Something a Little More Advanced

Now we're going to take an existing repo, pull it down, and spin up a fully functioning Drupal site. To do this, we're going to be using a modified fork of the Docksal Composer-based Drupal 8 repo.

### Setting up for the project

To begin, we need to clone the git repository, or "repo", at https://github.com/JDDoesDev/docksal-training/. We'll do this in the `projects` folder we used in the last section.

``` bash
$ cd ~/projects
$ git clone git@github.com:JDDoesDev/docksal-training.git
$ cd docksal-training
$ git checkout docksal-training/drupal-site-step-1
```

Here we're pulling the repo and checking out the Step 1 branch, where we're going to begin working on the project.

### Spinning Up Drupal

{{% notice info %}}
**NOTE:** There is a possibility that the default settings may not provision enough memory or CPU power in your virtual machine for some steps of the next section. If your build fails, then consult the [Troubleshooting](#troubleshooting) portion of this section.
{{% /notice %}}

Now that we've cloned the repo we're going to run a command we haven't talked about yet: `fin init`

Before we run that, a little info on what `fin init` is and what it does. `fin init` is a custom command that is not part of the core Docksal package. Instead, it is customized on a project by project basis, and sometimes is not required. The purpose of `fin init` is usually to completely start or restart a project from a clean state.

This means that when you run `fin init`, it will remove all existing project containers and volumes, recreate them, and run any other functions defined within the script.

Let's run `fin init` and see what happens.

``` bash
$ fin init
```

The output should start with

``` bash
Step 1  Initializing stack...
Removing containers...
Removing network docksal-training_default
WARNING: Network docksal-training_default not found.
Removing volume docksal-training_cli_home
WARNING: Volume docksal-training_cli_home not found.
Removing volume docksal-training_project_root
WARNING: Volume docksal-training_project_root not found.
Removing volume docksal-training_db_data
WARNING: Volume docksal-training_db_data not found.
Volume docksal_ssh_agent is external, skipping
```

The `WARNING:` lines let us know that the project does not already exist, but it also points out that we're attempting to remove the current containers as part of the `fin init` process.

If all runs according to plan you will see Composer do some things, Docksal do some things, and Drupal do some things, eventually ending with

``` bash
[notice] Starting Drupal installation. This takes a while.
[success] Installation complete.  User name: admin  User password: 2kEpnqm4dh
real 26.55
user 9.47
sys 3.42
Open http://docksal-training.docksal in your browser to verify the setup.
Look for admin login credentials in the output above.
DONE!  Completed all initialization steps.
```

Let's visit our newly created Drupal site at `http://docksal-training.docksal` and see what we have.

If everything went as expected, then you should see a vanilla Drupal 8 site with no content. However, there were a few things that needed to happen for us to get this spun up and functioning. If you've ever installed a Drupal site, you know that ordinarily we need to alter some database settings and customize other settings for Drupal to install. Let's take a look at where these things happened with Docksal.

### Examining `fin init`

The `fin init` command exists in two pieces, both of which are inside the `.docksal/commands` folder.

The files are `init` and `init-site`

``` bash
.docksal
├── commands
│   ├── init
│   └── init-site
```

When you put a file in the `.docksal/commands` folder, it tells Docksal that you want to make whatever is inside that file available as a command to this project, and this project only. The commands should be written as Bash scripts and it should be understood that the commands, by default, will be run from **outside** the container. This means that if you want something to happen **inside** the container, you need to wrap it in the `fin` command.

Inside our `.docksal/commands/init` file we'll notice a few things:

First, our shebang:

``` bash
#!/usr/bin/env bash
```

If you're unfamiliar, a `shebang` tells our system what it should use to run the remainder of the file. In this case, it's being told to run the file as interpreted by `bash`.

Further down, there are two commands that are being run: `fin project reset -f` and `fin init-site`.

* `fin project reset -f` does exactly what it says it does. It resets the project to a clean state by removing all containers and volumes and restarting them.

* `fin init-site` begins the script defined at `.docksal/commands/init-site`

### Examining `fin init-site`

The opening of `.docksal/commands/init-site` looks similar to `.docksal/commands/init`, but there is one line that makes a major difference:

``` shell
#: exec_target = cli
```

This tells Docksal that we will be running the commands inside the `cli` container and commands don't need to be wrapped in `fin`. This is extremely important to remember when customizing our project with our own commands.

The rest of the `init-site` file is comprised of the steps needed to install Drupal, including:

* Running `composer install`
* Copying our custom `settings.php` and `settings.local.php` to `web/sites/default/`
* Fixing permissions
* Installing Drupal using Drush

At the end of this script, it outputs how long it took to run, the URL of the new project, and the generated username and password combination.

### Summary

In this section we pulled a customized Drupal 8 boilerplate from a git repo and spun it up on our local dev environment with one command, `fin init`. We also tested that our site exists by visiting the generated URL. We should have an understanding of what `fin init` is for and how to use it to start or restart a project from a clean state.

We also examined how `fin init` uses `init-site` to build a project and the steps it goes through, including telling Docksal to run the commands within the `cli` container.

Next, we're going to customize our install and our site to make it a little less default.

## Troubleshooting

It is likely that your `fin init` may fail due to a Composer memory error. If you see output similar to the following:
``` bash
The following exception is caused by a lack of memory or swap, or not having swap configured
Check https://getcomposer.org/doc/articles/troubleshooting.md#proc-open-fork-failed-errors for details


  [ErrorException]
  proc_open(): fork failed - Cannot allocate memory

```

That means you'll need to add some resources to your virtual machine.

1. Run `fin system stop` to shut down the VM.
2. Open your VirtualBox application.
3. Highlight the `docksal` machine, making sure it is in a `Powered off` state.
4. Select "Settings".
5. Under "System > Motherboard" increase the memory to 4096 MB
6. Under "System > Processor" increase the CPUs to 4
7. Go back to your terminal and run `fin system start`
8. Retry `fin init`
