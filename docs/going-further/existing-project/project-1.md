---
title: "Adding Docksal to a Basic Project"
weight: 4
---

## Scenario

{{% notice info %}}
As lead developer on a team you've run into a lot of issues with your teammates submitting features for code review and you notice that your continuous integration testing fails frequently. When you pull down to test for yourself, you see a lot of code that was deprecated in a previous version of PHP and will break if run on the version of PHP that your hosting provider uses.</br>
After some conversations, you find out that one of your teammates is using MAMP while another is using native Apache and PHP 5.6 installed on their system. A third is using a Vagrant setup that hasn't been maintained in over a year.
{{% /notice %}}

This is not an uncommon scenario. While not every team should have a prescribed local dev environment, if time is lost because there are errors affecting deploying code efficiently, then it may be time to consider a strong recommendation.

Let's look at the following code that is valid in PHP 5.6:

``` php
<?php

$string = 'foo, bar, baz';

function get_array($string) {
  $array = split(', ', $string);

  return $array;
}

var_dump(get_array($string));

?>
```

If we were to run this using a system that has PHP 5.6 installed, which it shouldn't be, the only thing we might see is a deprecation warning.

```
Deprecated: Function split() is deprecated in /var/www/public/test.php on line 6
array(3) { [0]=> string(3) "foo" [1]=> string(3) "bar" [2]=> string(3) "baz" }
```

However, the `split()` PHP function was removed in 7.0. If we try running this same code on a server with PHP 7.0 or greater, we'd get this error:

```
Fatal error: Uncaught Error: Call to undefined function split() in /var/www/public/test.php:6 Stack trace: #0 /var/www/public/test.php(11): get_array('foo, bar, baz') #1 {main} thrown in /var/www/public/test.php on line 6
```

This can be frustrating to everyone involved as it is a source of the "But it worked on my machine!" problem. Let's get rid of this problem by adding Docksal to the project repo and allowing everyone else to reap the benefits of using a pre-set local development environment.

For this exercise we're going to create a new project and add Docksal to it.

1. In our `~/projects/` folder create the `add-docksal` folder.

    ``` bash
    $ cd ~/projects/
    $ mkdir add-docksal
    $ cd add-docksal
    ```

2. Create an `index.html` file.

    ``` bash
    $ touch index.html
    ```

1. Open this file in your editor and add the following:

    ``` html
    <html>
      <p>Hello, Docksal!</p>
    </html>
    ```

1. From your system's file explorer, open this file with your browser. If we wanted to, we could use an existing stack to setup our virtual hosts, our host file, and our web server. Depending on the existing setup, this could take a bit of time.

Now that we're able to view the rendered HTML, but if we wanted to start adding in any server-side code, like PHP or NodeJS, we would need to setup the back end of our web stack. Instead we're going to add Docksal to this project. We do that by doing three things.

1. Create a `docroot` folder inside your `~/projects/add-docksal/` folder and move `index.html` into it.

    ``` bash
    $ mkdir docroot
    $ mv index.html docroot/
    ```

1. Create a `.docksal` folder in your `~/projects/add-docksal/` folder.

    ``` bash
    $ mkdir .docksal
    ```

1. Run `fin up` and watch as your project gets spun up.

That's literally all you need to do to add Docksal to an existing project. If you look in your `~/project/add-docksal/.docksal/` folder now, you'll notice that Docksal has taken the liberty of adding your `docksal.env` and `docksal.yml` files. Inside the `docksal.env` it tells Docksal to use the default stack.

After our project spins up, we can add the changes to the project to our repo and everyone working on this project now has access to the exact same development environment.

If you remember, Docksal looks for your web application files in the `docroot` folder. This is why we created this folder and moved our `index.html` file into it before spinning up.

### Summary

The process for adding Docksal to an existing project is simple and straightforward. If you have Docksal installed on your system, all that you need to do is add a `.docksal` folder to your project root and make sure that the files your server needs to render are inside the `docroot` folder. Once you have these done, all you have to do is run `fin up` and Docksal takes care of the rest.

{{% notice tip %}}
The finished code for this project is available at https://github.com/JDDoesDev/docksal-training-projects/tree/adding-docksal-project-1.
{{% /notice %}}
