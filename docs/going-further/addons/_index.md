---
title: Addons
weight: 5
---

## Using Docksal addons to make development easier

We touched on creating your own commands in [Advanced Customizations: Project 3](/docksal-training/going-further/advanced-customizing/project-3/), however many frequently used commands have been contributed to the Docksal project in the form of addons.

### List of current addons

|   Name	|  Description 	|  Requirements 	|
|--- |--- |--- |
|   adminer | [Adminer](https://www.adminer.org/) database management tool | MySQL |
|   andock | [Andock](https://andock.readthedocs.io/en/latest/) makes it dead simple to get Docksal environments up on your server. | Docksal |
|   artisan | Runs [Laravel's Artisan](https://laravel.com/docs/artisan) command in `cli`. **Requires** artisan pre-installed inside `cli`. | Laravel, Artisan |
|   blt | Acquia BLT tool launcher (requires [BLT installation](https://blog.docksal.io/docksal-and-acquia-blt-1552540a3b9f)) | Drupal |
|   codeclimate | [CodeClimate](https://codeclimate.com/) code quality tool | |
|   mailhog | [Mailhog](https://github.com/mailhog/MailHog) e-mail capture service for current project |  |
|   phpcs | PHP Code Sniffer and Code Beautifier | |
|   phpunit | Creates a phpunit.xml file and runs PHPUnit tests | Drupal |
|   pma | [PhpMyAdmin](https://www.phpmyadmin.net/) database management tool | MySQL |
|   redis | Add [Redis](https://redis.io/) to current project |  |
|   sequelpro | Launches [SequelPro](https://www.sequelpro.com) with the connection information for current project. | macOS |
|   simpletest | Runs SimpleTest tests in Drupal 7 and 8 | Drupal |
|   solr | [Apache Solr](http://lucene.apache.org/solr/) search service for current project |  |
|   uli | Generate one time login url for current site | Drupal |

All of these are available to install and use to make development a little quicker and easier. Let's install an addon.

We're going to install `uli` which is a wrapper for the Drupal Drush command `drush uli`.

``` bash
$ fin addon install uli
```

This will download the file to your `.docksal/addons` folder and make it available to your project.

To run any addon, you just need to run `fin [addon name]`. For example:

``` bash
$ fin uli
```

Additionally, since Docksal is an open source project, you can create your own addons and submit them back to the project. Say you have a command that speeds up your process and you think others may benefit from it. To contribute back, you would do the following:

1. Fork the addons repo from https://github.com/docksal/addons
2. Create a feature branch
3. Add your addon into your fork of the repo
4. Update the repo's `README.md` file with your addon a description and any requirements
5. Submit a pull request against the Docksal addons repo with a description of what your addon does and how it's beneficial

### Summary

Addons are generally community contributed commands that have been submitted to the Docksal project for the greater good of all users. You can use existing addons or submit your own back to the project. The process for installing an addon is simple and after an addon is in your project, you are free to tweak it to your own needs.
