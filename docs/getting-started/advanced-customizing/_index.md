---
title: "Advanced Customizations"
weight: 7
---

## Truly Making a Project Custom

In the last section we customized a few things to change our domain name and the site name. In this section, we're going to show how to extend a stack, change versions of images, and even customize and extend Docksal images using a Dockerfile.

### Project 1: Configuring to Match Production

For this project, we're going to use the following scenario:

{{% notice info %}}
Your client has a site on Acquia using PHP 7.3. You know that the default Docksal Acquia stack uses PHP 7.2, but you want to match environments. How do you do this?
{{% /notice %}}

1. **Choose the right stack**

    We're going to start by configuring Docksal to use the Acquia stack. In your `docksal.env` file, find the line `DOCKSAL_STACK=pantheon` and change it to `DOCKSAL_STACK=acquia`.

1. **Configure the docroot**

    Acquia projects require that the Drupal installation live in `docroot` instead of `web`. This is the default webroot for Docksal, but to remain verbose we're going to change the environment variable. Find the line `DOCROOT=web` and change it to `DOCROOT=docroot`

1. **Update docroot and composer.json**

    Since our projects have been using the `web` folder for the docroot so far, we need to rename it. This is most easily accomplished in an IDE or in your system's version of a file explorer, however you can do this on a macOS or Linux terminal by using the `mv` command. `mv web docroot`. In addition, we also need to update our `composer.json` file to point to the correct folders.

    In `composer.json`, in the `extra.installer-paths` section, you need to change all instances of `web/` to `docroot/`. Example: `"web/core": ["type:drupal-core"],` becomes `"docroot/core": ["type:drupal-core"],`

2. **Configure the PHP version**

    The PHP version is defined in the `cli` service. The default image for the `cli` service is `docksal/cli:2.6-php7.2`. We can change this in `docksal.env` by adding the following variable: `CLI_IMAGE=docksal/cli:2.9-php7.3`. This is the latest version of this image tagged with PHP 7.3.

    Inside `~/.docksal/stacks/services.yml` the `cli` section runs logic for the image version: `image: ${CLI_IMAGE:-docksal/cli:2.6-php7.2}` which checks to see if the `CLI_IMAGE` environment variable is set, and if not, uses the default.

3. **Update your project**

    The easiest way to do this is to run `fin up`, however if you have not initialized your project yet, you should run `fin init`. For this exercise we're going to run `fin init`.

Run `fin init` and watch as the images that aren't on your system are pulled down and your project spins up using PHP 7.3.x.

{{% notice tip %}}
The completed code for Project 1 can be found at https://github.com/JDDoesDev/docksal-training-projects/tree/adv-cust-project-1
{{% /notice %}}

### Project 2: The Front-end Team Requires a Specific Version of NPM and NodeJS

For this project, we're going to use the following scenario:

{{% notice info %}}
The front-end team is using a theme that is shared across multiple projects as a starter-kit. In order to reduce compatibility errors, they have requested that the Docksal setup accounts for this by locking to a specific version of NodeJS and NPM so that they don't have to completely overhaul the NPM dependencies.
{{% /notice %}}

To accomplish this, we're going to do a couple of things in order to lock a version of NodeJS and NPM.

1. **Lock the versions in a custom Dockerfile**

    We're going to create a custom Dockerfile that extends the default `cli` image within our project.

    1. Create the Dockerfile at `.docksal/services/cli/Dockerfile`
    2. Extend the current image by starting the file with

        ``` dockerfile
        FROM docksal/cli:2.9-php7.3
        ```

        This tells Docksal that we're still going to use this image, but we're doing something more with it.

    1. Change the Dockerfile user so that settings aren't changed as `root`

        ``` dockerfile
        USER docker
        ```

    1. Run commands that will install and lock the version of NodeJS and NPM.

        ``` dockerfile
        RUN set -e; \ # Note the ';' and the '\'.
          # Initialize the user environment (this loads nvm)
          . $HOME/.profile; \
          # Install the necessary nodejs version and remove the unnecessary version
          nvm install 8.11.0; \
          nvm alias default 8.11.0; \
          nvm use default; \
          # Install packages
          npm install -g npm@6.1.0; \
          # Cleanup
          nvm clear-cache && npm cache clear --force; \
          # Fix npm complaining about permissions and not being able to update
          sudo rm -rf $HOME/.config
      ```

        Notice that every command in the `RUN` directive is followed with `; \` except the last line. This is because `RUN` directives are concatinated into a single line when run.

        We're sourcing the `~/.profile` file and getting our NVM (Node Version Manager) alias early on so that we can choose our version. In this case, we're using 8.11.0. Following that, we set this version as our default.

        Next, we install our locked version of NPM, clean up our caches, and then run some permission fixes, but we're not done yet.

    2. Return control to the `root` user.

      ``` docker
      USER root
      ```

          Our completed Dockerfile should look like this

          ```dockerfile
          FROM docksal/cli:2.9-php7.3

          USER docker

          # Install additional global npm dependencies
          RUN set -e; \
              # Initialize the user environment (this loads nvm)
              . $HOME/.profile; \
              # Install the necessary nodejs version
              nvm install 8.11.0; \
              nvm alias default 8.11.0; \
              nvm use default; \
              nvm uninstall 10.16.3; \
              # Install packages
              npm install -g npm@6.1.0; \
              # Cleanup
              nvm clear-cache && npm cache clear --force; \
              # Fix npm complaining about permissions and not being able to update
              sudo rm -rf $HOME/.config

          USER root
          ```

2. **Call the custom Dockerfile**

        In `docksal.yml` we're going to define our `cli` service.


        1. Open `.docksal/docksal.yml`
        2. Add the following to the `cli:` section:

            ```yaml
            cli:
              image: ${COMPOSE_PROJECT_NAME_SAFE}_cli
              build: ${PROJECT_ROOT}/.docksal/services/cli
            ```

            This names the image based on our project's name and then tells Docksal where to find the file we're going to build the new image from.

3. **Update the project**

        Instead of running `fin init` here, we're going to run `fin up`. Keep an eye on the output as the new image is built.

        ``` shell
        $ fin up
        Starting services...
        Building cli
        Step 1/4 : FROM docksal/cli:2.9-php7.3
        ---> bef7b0b7014f
        Step 2/4 : USER docker
        ---> Using cache
        ---> d563a10b8db0
        Step 3/4 : RUN set -e;   . $HOME/.profile;   nvm install 8.11.0;   nvm alias default 8.11.0;   nvm use default;   npm install -g npm@6.1.0;   nvm clear-cache && npm cache clear --force;   sudo rm -rf $HOME/.config
        ---> Running in d503e299f557
        /bin/sh: 39: /home/docker/.profile: [[: not found
        Downloading and installing node v8.11.0...
        Downloading https://nodejs.org/dist/v8.11.0/node-v8.11.0-linux-x64.tar.xz...
        ######################################################################## 100.0%
        Computing checksum with sha256sum
        Checksums matched!
        Now using node v8.11.0 (npm v5.6.0)
        default -> 8.11.0 (-> v8.11.0)
        Now using node v8.11.0 (npm v5.6.0)
        Uninstalled node v10.16.3
        /home/docker/.nvm/versions/node/v8.11.0/bin/npx -> /home/docker/.nvm/versions/node/v8.11.0/lib/node_modules/npm/bin/npx-cli.js
        /home/docker/.nvm/versions/node/v8.11.0/bin/npm -> /home/docker/.nvm/versions/node/v8.11.0/lib/node_modules/npm/bin/npm-cli.js
        + npm@6.1.0
        added 247 packages, removed 41 packages and updated 129 packages in 25.413s
        nvm cache cleared.
        npm WARN using --force I sure hope you know what you are doing.
        Removing intermediate container d503e299f557
        ---> 4d9ec2ec20f4
        Step 4/4 : USER root
        ---> Running in 70ebd3d709ff
        Removing intermediate container 70ebd3d709ff
        ---> 1c70700b6839
        Successfully built 1c70700b6839
        Successfully tagged my-first-docksal-project_cli:latest
        ```

        Every step in the Dockerfile is tagged in the image with a hash.

        ``` bash
        Step 1/4 : FROM docksal/cli:2.9-php7.3
        ---> bef7b0b7014f
        ```

        The hash `bef7b0b7014f` indicates a layer of the image. If something were to break when building the image, this gives us a point of reference to examine the image and see what happened. As you can see, our image built successfully and our system now has the correct versions of NodeJS and NPM installed. We can check this by running the following:

        ``` bash
        $ fin exec node -v
        v8.11.0

        $ fin exec npm -v
        6.1.0
        ```

    Now our front-end team has the version they need installed, and everyone is happy.

{{% notice tip %}}
The completed code for Project 2 can be found at https://github.com/JDDoesDev/docksal-training-projects/tree/adv-cust-project-2
{{% /notice %}}

### Project 3: A Multi-server Project

For this lesson we're going to look at the following scenario:

{{% notice info %}}
A client wants a decoupled solution to be hosted on two servers. The Drupal side will be hosted on Acquia and the front-end side will be hosted on a different server. The project is going to use GatsbyJS for the front-end.
{{% /notice %}}

Now, we can build off our last step by continuing to grow our `docksal.yml` and `docksal.env` files. We're also going to add in some custom commands to make our life a little bit easier in the long run.

Let's start off by listing what we're going to needing for this project:

* A Drupal installation on an Acquia Stack
* Multiple domains
* Two different simulated servers
* The ability to connect the two servers
* NodeJS
* npm
* Gatsby CLI

Now, we could simulate the different servers by simply using multiple VirtualHosts in Apache, but that still makes it a little too easy for the Drupal and Gatsby side to communicate. To overcome this, we're going to simulate by keeping the Gatsby server in its own service, separate from the Drupal site.

However, we can still use a shared codebase so that we only need to maintain a single git repo.

Let's get started:

1. **Create a Custom Gatsby Service - Dockerfile**

        To do this we're going to use both a custom Dockerfile and edit our `docksal.yml`. Let's begin by creating the custom Dockerfile.

        1. Create the file `.docksal/services/gatsby/Dockerfile`
        2. We're going to extend the default Docksal CLI image so start the file with:

            ```dockerfile
            FROM docksal/cli:2.9-php7.3
            ```

        3. We want to run all of our installations as our default container user "docker" so we need to make sure to switch users in the Dockerfile.

            ``` dockerfile
            USER docker
            ```

        4. Now we want to ensure that the commands run in Bash instead of the default shell. We need to tell the image to build that way.

          ``` dockerfile
          SHELL ["/bin/bash", "-c"]
          ```

            This tells Docker to build this layer of the image as though it were in a Bash shell. The flag `-c` tells it to get ready for the command.

        5. This next section should look pretty familiar. We're going to tell the image to add in a locked version of NodeJS, a locked version of NPM, and this time we're adding in Gatsby CLI.

            ``` dockerfile
            RUN set -e; \
              # Initialize the user environment (this loads nvm)
              source $HOME/.profile; \
              # Install the necessary nodejs version
              nvm install 10.15.0; \
              nvm alias default 10.15.0; \
              nvm use default; \
              # Install packages
              npm install -g npm@6.4.1; \
              npm install -g gatsby-cli; \
              # Cleanup
              nvm clear-cache && npm cache clear --force; \
              # Fix npm complaining about permissions and not being able to update
              sudo rm -rf $HOME/.config;
            ```

              The biggest changes here from our last project are that we're locking NodeJS at 10.15.0, we're installing npm 6.4.1, and we're installing Gatsby CLI.

        1. We need to return to the default shell and switch back to the `root` user.

            ``` dockerfile
            SHELL ["/bin/sh", "-c"]

            USER root
            ```

              Just like our earlier `SHELL` directive, we're running the command to change back to `sh`, the default shell, and then switching to the `root` user for further commands.

        1. Finally, we need to tell our service to expose the port that the development server for Gatsby runs on. In this case, `8000`.

            ``` dockerfile
            EXPOSE 8000
            ```

        The final Dockerfile should look like this:

        ``` dockerfile
        FROM docksal/cli:2.9-php7.3

        USER docker

        SHELL ["/bin/bash", "-c"]
        # Install additional global npm dependencies
        RUN set -e; \
            # Initialize the user environment (this loads nvm)
            source $HOME/.profile; \
            # Install the necessary nodejs version
            nvm install 10.15.0; \
            nvm alias default 10.15.0; \
            nvm use default; \
            # Install packages
            npm install -g npm@6.4.1; \
            npm install -g gatsby-cli; \
            # Cleanup
            nvm clear-cache && npm cache clear --force; \
            # Fix npm complaining about permissions and not being able to update
            sudo rm -rf $HOME/.config;

        SHELL ["/bin/sh", "-c"]

        USER root

        EXPOSE 8000
        ```

        Save the Dockerfile and close it out.

2. **Create a Custom Gatsby Service - `docksal.yml`**

        Now that we have our Dockerfile created, we need to tell Docksal to use it when putting together the application. For that, we need to open `.docksal/docksal.yml` and make some changes.

        Remember, indentation matters in a YAML file.

        1. In our `docksal.yml` file we're going to define a new service. This will go under the parent `services` and should have the same indentation as our customized `cli` service. We're going to name this service `gatsby`.

            ``` yaml
            version: "2.1"
            services:
              cli:
                ...
              gatsby:
            ```

        2. In order to function within our application, there are a few things that we need to pass to the `gatsby` service. The host user ID, the host group ID, and the DNS information. We can pass the host user and group IDs through the `environment` component and the DNS settings through the DNS component.

            ``` yaml
            gatsby:
              environment:
                - HOST_UID
                - HOST_GID
              dns:
                - ${DOCKSAL_DNS1}
                - ${DOCKSAL_DNS2}
            ```

            The `DOCKSAL_DNS1` and `DOCKSAL_DNS2` variables are defined in the `fin` binary based on the local IP address of the Docksal VM and the remote DNS server `8.8.8.8`

        3. Now we need to tell Docksal a little more about our service. We're going to do this by defining the hostname, the image name, and where to build the image from. We can do this by adding a few more items to our `docksal.yml`. For consistency with other services, we're going to place these immediately following the `gatsby:` declaration.

            ``` yaml
            gatsby:
              hostname: gatsby
              image: ${COMPOSE_PROJECT_NAME_SAFE}_gatsby
              build: ${PROJECT_ROOT}/.docksal/services/gatsby
              environment: ...
            ```

            The hostname is what we can use to identify this service without having to type the entire service name out.

            Also, notice that we did _not_ include `Dockerfile` in our build. This is on purpose because Docker knows we're looking for a Dockerfile.

        4. It's time to tell Docksal what domain to use for this service. We're going to do this by using `labels`. We'll be using the `io.docksal.virtual-host`, `io.docksal.virtual-port`, and `io.docksal.cert-name` labels for this.

            One thing to note is that when we run the command to start the development server for Gatsby, it's going to start a NodeJS process so the domain will point directly to this service and load what is being served by NodeJS.

            Add the following below your `build:` line:

            ``` yaml
            labels:
              - io.docksal.virtual-host=gatsby.${VIRTUAL_HOST}
              - io.docksal.virtual-port=8000
              - io.docksal.cert-name=${VIRTUAL_HOST_CERT_NAME:-none}
            ```

            These set the domain, which will be `gatsby.mycustomsite.docksal`, the port that this domain will point to, and the cert name if we want or need to simulate an SSL environment.

        5. Next, we need to tell the service where it should look for files to mount a volume. We'll do this using `volumes:` and point specifically to the `gatsby` folder.

            Below your `labels` put in the `volumes:` information

            ``` yaml
            volumes:
               - ${PROJECT_ROOT}/gatsby:/var/www/gatsby
               - ${SSH_AUTH_SOCK:-docksal_ssh_agent}:${SSH_AUTH_SOCK:-/.ssh-agent}:ro
            ```

            We're pointing this service to create an unnamed volume mounted to our `gatsby` folder that will exist as `/var/www/gatsby` within the container. We're also passing along our SSH keys in case we need them for anything.

        6. Now some final touches to make life easier. We're going to add in a `working_dir` that makes it easier for us to run commands in our container, and a little environment variable that allows for us to watch for file changes, which is very handy when working with something using any kind of live-reload functionality.

            We do this because the NFS bind we're using for our volumes does _not_ track file changes and send notifications. Instead, we're using [Chokidar](https://www.npmjs.com/package/chokidar), an NPM package that simulates filesystem events.

            Add the following to your `docksal.yml`:

            ``` yaml
            environment:
              - HOST_UID
              - HOST_GID
              - CHOKIDAR_USEPOLLING=1 # <== New Line
            working_dir: /var/www/gatsby # <== New Line
            ```

        Our `docksal.env` should look like this now:

        ``` yml
        version: "2.1"
        services:
          cli:
            image: ${COMPOSE_PROJECT_NAME_SAFE}_cli
            build: ${PROJECT_ROOT}/.docksal/services/cli
            environment:
              - COMPOSER_MEMORY_LIMIT
              - SITE_NAME
          gatsby:
            hostname: gatsby
            image: ${COMPOSE_PROJECT_NAME_SAFE}_gatsby
            build: ${PROJECT_ROOT}/.docksal/services/gatsby
            labels:
              - io.docksal.virtual-host=gatsby.${VIRTUAL_HOST}
              - io.docksal.virtual-port=8000
              - io.docksal.cert-name=${VIRTUAL_HOST_CERT_NAME:-none}
              - io.docksal.shell=bash
            volumes:
              - ${PROJECT_ROOT}/gatsby:/var/www/gatsby
              - ${SSH_AUTH_SOCK:-docksal_ssh_agent}:${SSH_AUTH_SOCK:-/.ssh-agent}:ro
            environment:
              - HOST_UID
              - HOST_GID
              - CHOKIDAR_USEPOLLING=1
            working_dir: /var/www/gatsby
            dns:
              - ${DOCKSAL_DNS1}
              - ${DOCKSAL_DNS2}
        ```

        But don't close it out yet. There's a bit more to do here.

3. **Add a Custom Web Service for the Static Build**

        In order to fully simulate a two server setup, we want to let our Gatsby static site have its very own web server. We're going to do this by creating a custom web service that will handle the requests to the static site and route them to the Gatsby service and volume.

        1. Start by creating another service in your `docksal.yml` file. Call it `gatsby_web`. This should be the same indentation as your other services.

            ``` yaml
              gatsby_web:
            ```

        2. We're going to use the `docksal/apache:2.4-2.3` image for this so let's add that image to our service.

            ``` yaml
            gatsby_web:
              image: docksal/apache:2.4-2.3
            ```

        3. Next, tell Apache where to point the webserver by creating a volume for the server.

            ``` yaml
              gatsby_apache:
                image: docksal/apache:2.4-2.3
                volumes:
                  - ${PROJECT_ROOT}/gatsby:/var/www/gatsby
            ```

        4. We need to set an environment variable to tell apache where the docroot of our project is, otherwise it defaults to `/var/www/docroot`. We'll set that next.

            ``` yaml
                environment:
                  - APACHE_DOCUMENTROOT=/var/www/gatsby/public
            ```

        5. Finally, we're going to assign the domain using a label, much like we did with our `gatsby` service.

            ``` yaml
                labels:
                  - io.docksal.virtual-host=static.${VIRTUAL_HOST}
            ```

        Our new service should look like:

        ``` yaml
          gatsby_apache:
            image: docksal/apache:2.4-2.3
            volumes:
              - ${PROJECT_ROOT}/gatsby:/var/www/gatsby
            environment:
              - APACHE_DOCUMENTROOT=/var/www/gatsby/public
            labels:
              - io.docksal.virtual-host=static.${VIRTUAL_HOST}
        ```

        We are not going to cover building a decoupled project here, but to test this you could create an `index.html` inside the `gatsby\public` folder, run `fin project start` because `fin up` may not capture all of your changes, and visit `static.mycustomsite.docksal` to see it load.

{{% notice tip %}}
The completed code for Project 3 can be found at https://github.com/JDDoesDev/docksal-training-projects/tree/adv-cust-project-3
{{% /notice %}}

### Summary

In this section we went through many of the advanced customizations we can do with Docksal services using `docksal.yml`, `docksal.local`, and even a couple of custom Dockerfiles. We created a two server application, changed hosting providers, and locked down versions of certain tools to keep our team happy. As you can see, Docksal is an extremely flexible and powerful tool.

Next, we're going to look into using local files for settings and variables that should not live in the repo.
