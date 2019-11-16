---
title: Adding Docksal to a Drupal Project
weight: 3
---

## Scenario

{{% notice info %}}
You have several clients and projects that you do work for. You've been using a custom AMP (Apache, MySQL, PHP) stack, but you're frustrated with having to change PHP versions for each client project. You've decided to use Docksal for a local development environment, but you don't want to lose all of the work you've done.
{{% /notice %}}

Let's take a moment to consider what we definitely need for our site to work on a Docksal project.

* Apache, MySQL, PHP
* A database connection
* Drush
* Composer
* A database backup


{{% notice tip %}}
The starting code for this project is available at https://github.com/JDDoesDev/docksal-training-projects/tree/adding-docksal-project-2-start
{{% /notice %}}

1. The first thing we need is a backup of our database. How we do this depends on how your project is setup.

    **Option 1: Sync from production**

    If your project is already hosted and you have Drush aliases setup to your production site, you'll be able to import the database by using a command like `drush sql-sync @remote @local` depending on your Drush aliases.

    **Option 2: Create a local dump**

    If your project is only local and you don't have a remote to sync from, you can create a database dump by using `drush sql:dump` before shutting down your existing project and importing after we get setup.


1. Next, we need to add Docksal to our project. Assuming that we're working in our `~/projects/docksal-training-projects` folder on the branch `adding-docksal-project-2-start` we'll need to create the `.docksal` folder.

    ``` bash
    $ mkdir .docksal
    ```

    Now we have Docksal as part of our project, but we're missing a few things. For one, we don't have anything telling Drupal where to find the database. This usually lives in `docroot/sites/default/settings.php`.

{{% notice info %}}
It is normally considered bad practice and insecure to include your live `settings.php` file in the repo, however this demo does not contain any sensitive data or real passwords.
{{% /notice %}}

1. We don't want to override our production settings with our Docksal settings, so we're going to create a `docksal.settings.php` file that we will conditionally require in our `settings.php`

    ``` bash
    $ cd docroot/sites/default/
    $ touch docksal.settings.php
    ```

    Open this file in your text editor.

1. Add Docksal required settings to your `docksal.settings.php`.

    ``` php
    <?php // Don't forget the opening php tag
    # Docksal DB connection settings.
    $databases['default']['default'] = [
      'database' => 'default',
      'username' => 'user',
      'password' => 'user',
      'host' => 'db',
      'driver' => 'mysql',
    ];
    ```

    The following is for Drupal 8 only. If you are using Drupal 7 then replace `$settings` with `$conf` in the following lines.

    ``` php
    # Workaround for permission issues with NFS shares in Vagrant
    $settings['file_chmod_directory'] = 0777;
    $settings['file_chmod_file'] = 0666;
    ```

1. Reset your permissions on your `docroot/sites/default/files` folder to make sure your files are accessible to your containerized project.

    From your `docroot/sites/default` folder run the following:

    ``` bash
    $ chmod -R +rwX files
    ```

    This command uses `chmod` to change the mode of the files and folders. `-R` is for recursive. The additional flags are `r` for 'read', `w` for 'write', and `X` indicates to make folders or directories executable/searchable (notice the capital `X` instead of `x`). This sets the folders to `777` or `rwx` and files to `666` or `rw`.

1. Tell Drupal where to find our `docksal.settings.php` file.

    To do this we have a couple of options. We can either uncomment the section at the bottom of the default `settings.php` file

    ``` php
    if (file_exists($app_root . '/' . $site_path . '/settings.local.php')) {
      include $app_root . '/' . $site_path . '/settings.local.php';
    }
    ```

    and replace `settings.local.php` with `docksal.settings.php`, but this will cause some issues if we forget to ignore our `docksal.settings.php` file in version control.

    My preferred method is to use the following snippet:

    ``` php
    // Signals that this is operating in a Docksal environment and should include
    // default Docksal settings.
    if (getenv('PROJECT_ROOT') && file_exists($app_root . '/' . $site_path . '/docksal.settings.php')) {
      require $app_root . '/' . $site_path . '/docksal.settings.php';
    }
    ```

    The reason I prefer this snippet is because instead of just checking if the file `docksal.settings.php` exists, it also checks to see if the `PROJECT_ROOT` environmental variable exists, which it always will in a Docksal project, and then includes `docksal.settings.php`.

1. Start Docksal with `fin project start`

    **Troubleshooting Tip**

    If you've been working in this folder on other projects from this training, you may run into an issue where the volumes are confused. If you try to run a command and get an error saying that the file cannot be found or similar, then you may need to do the following steps to clear any existing volumes and containers:

    1. Run `fin project remove`
    1. Run `fin system restart`
    1. Run `fin project start`

2. If you cloned this project from version control, install the composer dependencies.

    ``` bash
    $ fin composer install
    ```

1. Create the default database for our project.

    ``` bash
    $ fin db create default
    ```

2. Visit your site at `docksal-training-projects.docksal`


If all went well, you should see the default install page. At this point we would import our database from our dump using `fin db import [file]` or from our production server using `fin drush`.

{{% notice tip %}}
The finished code for this project is available at https://github.com/JDDoesDev/docksal-training-projects/tree/adding-docksal-project-2-finish
{{% /notice %}}

### Summary

By using the default stack that Docksal imports to an empty `.docksal` folder, we have our Apache, MySQL, and PHP installed in our project. We told Drupal where to find our database, and composer and Drush are installed in our project by Docksal by default.

Adding Docksal to an existing Drupal project takes a few extra steps, but is pretty straightforward. With some practice you'll know what steps to take without having to refer to this training. You could even automate the process by creating your own `fin init` script to build the site, import the database from production, import config, and create the correct settings files. Next up we're going to explore some of the many addons that are available through Docksal.

## Troubleshooting

Some common issues when moving from a locally hosted project using WAMP, MAMP, or other containerized environments:

### Conflicting Ports

``` bash
Resetting Docksal services...
 * proxy
docker: Error response from daemon: driver failed programming external connectivity
on endpoint docksal-vhost-proxy (a7addf7797e6b0aec8e3e810c11775eb77508c9079e375c083b3650df2dff9a2):
Error starting userland proxy: listen tcp 0.0.0.0:443: listen: address already in use.
```

If Apache is running, it listens on the same ports (80, 443) as Docksal. Solve this by stopping Apache or reconfiguring Apache ports.

If Apache is not running, there may be some conflicts between Docksal and another local dev system. Run run `lsof -PiTCP -sTCP:LISTEN` and see what is using port 80 or port 443 and stop that process.

For other troubleshooting tips, visit the [Troubleshooting section of the Docksal docs](https://docs.docksal.io/troubleshooting/).
