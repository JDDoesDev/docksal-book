---
title: "Project 1: Configuring to Match Production"
weight: 8
---


## Scenario

For this project, we're going to use the following scenario:

{{% notice info %}}
Your client has a site on Acquia using PHP 7.3. You know that the default Docksal Acquia stack uses PHP 7.2, but you want to match environments. How do you do this?
{{% /notice %}}

### Choose the right stack

We're going to start by configuring Docksal to use the Acquia stack. In your `docksal.env` file, find the line `DOCKSAL_STACK=pantheon` and change it to `DOCKSAL_STACK=acquia`.

### Configure the docroot

Acquia projects require that the Drupal installation live in `docroot` instead of `web`. This is the default webroot for Docksal, but to remain verbose we're going to change the environment variable. Find the line `DOCROOT=web` and change it to `DOCROOT=docroot`

### Update docroot and composer.json

Since our projects have been using the `web` folder for the docroot so far, we need to rename it. This is most easily accomplished in an IDE or in your system's version of a file explorer, however you can do this on a macOS or Linux terminal by using the `mv` command. `mv web docroot`. In addition, we also need to update our `composer.json` file to point to the correct folders.

In `composer.json`, in the `extra.installer-paths` section, you need to change all instances of `web/` to `docroot/`. Example: `"web/core": ["type:drupal-core"],` becomes `"docroot/core": ["type:drupal-core"],`

### Configure the PHP version

The PHP version is defined in the `cli` service. The default image for the `cli` service is `docksal/cli:2.6-php7.2`. We can change this in `docksal.env` by adding the following variable: `CLI_IMAGE=docksal/cli:2.9-php7.3`. This is the latest version of this image tagged with PHP 7.3.

Inside `~/.docksal/stacks/services.yml` the `cli` section runs logic for the image version: `image: ${CLI_IMAGE:-docksal/cli:2.6-php7.2}` which checks to see if the `CLI_IMAGE` environment variable is set, and if not, uses the default.

### Update your project

The easiest way to do this is to run `fin up`, however if you have not initialized your project yet, you should run `fin init`. For this exercise we're going to run `fin init`.

Run `fin init` and watch as the images that aren't on your system are pulled down and your project spins up using PHP 7.3.x.

{{% notice tip %}}
The completed code for Project 1 can be found at https://github.com/JDDoesDev/docksal-training-projects/tree/adv-cust-project-1
{{% /notice %}}
