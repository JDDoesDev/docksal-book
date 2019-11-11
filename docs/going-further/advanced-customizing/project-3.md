---
title: "Project 3: Building a Multi-server Project"
weight: 10
---

### Scenario

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

Now, we could simulate the different servers by simply using multiple VirtualHosts in Apache, but that's not really simulating separate servers. Instead, it's using the same web server to host projects in different locations. To overcome this, we're going to simulate by keeping the Gatsby web server in its own service, separate from the Drupal site.

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

    6. We need to return to the default shell and switch back to the `root` user.

        ``` dockerfile
        SHELL ["/bin/sh", "-c"]

        USER root
        ```

          Just like our earlier `SHELL` directive, we're running the command to change back to `sh`, the default shell, and then switching to the `root` user for further commands.

    7. Finally, we need to tell our service to expose the port that the development server for Gatsby runs on. In this case, `8000`.

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

4. **Create a custom command to run commands inside our new service**

        It should be noted that normal `fin exec` commands will not run anything inside this service. If you need to run a command in a custom `cli`-type service, you'll need to add a few things to the command or create a custom command. An example of this would be running `gatsby develop`. Also, running `fin bash` will not get you into the service. For both of these commands you need to tell Docksal which service you are addressing.

        For `fin exec` we'll need the `--in` flag, and for `fin bash` we need to add the service name as an argument.

        Example:

        ``` bash
        $ fin exec --in=gatsby gatsby develop
        ```

        or

        ``` bash
        $ fin bash gatsby
        ```

        If you run these commands now, you may notice some things. One, the `fin exec` example doesn't work, and two, the `fin bash` example gets you to a sh shell instead of bash.

        By default, only the `cli` and `db` services have access to bash with `fin exec`. To get around this we need to wrap our commands with a few things. First we need to tell Docksal what user we want to run the commands as. Remember, when we created our Dockerfiles we created the user `docker`. Now we need to tell Docksal to run the command with this user and we do so by passing a variable to the command.

        Example:

        ``` bash
        $ container_user="-u docker" fin exec --in=gatsby
        ```

        The `container_user` variable is used in the `fin` binary to add arguments to the Docker commands that make Docksal work, which is why it's `-u docker` and not just `docker`.

        However, this does not give us access to the bash shell. What we need to do is tell Docksal to run this command in bash. We do that by actually executing the `bash` executable.

        ``` bash
        $ container_user="-u docker" fin exec --in=gatsby bash -lc "gatsby develop"
        ```

        The command here is telling `fin` to `exec`ute, in the container `gatsby` the `bash` executable. We pass in the `-l` flag to act as though it were invoked by a login shell, which gives us access to the aliases and commands in the `.bash_profile` and the `-c` flag to let `bash` know that we're giving it a command to run. Finally, we're giving it the command `"gatsby develop"`.

        This can be a bit unwieldy to type so we could create an alias for it, but to make it reusable for others who may be working on this project, we want to add it to the repo somehow. We can do this by defining a custom command.

        1. In your `.docksal/commands` folder create the file `gatsby-develop`. Go back to your project root, `~/projects/docksal-training-projects/`, and create this file.

            ``` bash
            $ cd .docksal/commands
            $ touch gatsby-develop
            $ chmod +x gatsby-develop
            ```

            In this example we create the file and then we make it executable by running `chmod +x`. This is necessary for it to run as a command, although Docksal is smart enough to try and fix it for you if you forget.

        2. In your editor, open this file up and let's add some things to it.

            Since it's a bash script, we need our shebang:

            ``` bash
            #!/usr/bin/env bash
            ```

            We also need our command:

            ``` bash
            container_user="-u docker" fin exec --in=gatsby bash -lc "gatsby develop"
            ```

            Now save this file and go back to your project root.

        3. Run the new command:

            ``` bash
            $ fin gatsby-develop
            ```

            Docksal will now pass in the command to the service and attempt to run `gatsby develop`, however it will error because we never installed Gatsby in this exercise.

5. **Create a command to shell into your `gatsby` service**

        We also want an entry point so that we can run more than just one command. For our default `cli` service, we can just run `fin bash`, but that will not work for our custom `gatsby` service. Instead, we're going to create the `gatsby-bash` command.

        1. In your `.docksal/commands` folder create the file `gatsby-bash`.

            Go back to your project root, `~/projects/docksal-training-projects/`, and create this file.

            ``` bash
            $ cd .docksal/commands
            $ touch gatsby-bash
            $ chmod +x gatsby-bash
            ```

        1. In your editor, open this file up and create the command.

            Since it's a bash script, we need our shebang:

            ``` bash
            #!/usr/bin/env bash
            ```

            We also need our command:

            ``` bash
            container_user="-u docker" fin exec --in=gatsby bash -il
            ```

            This time we're not passing a command, but we're using the `-i` flag instead of the `-c` flag. The `-i` flag indicates that we want to run this as an interactive shell and we also want it to have all of the commands from our `.bash_profile` file.

            Now save this file and go back to your project root.

        1. From your project root run the new command:

            ``` bash
            $ fin gatsby-bash
            ```

            You will now be inside your service, able to run any commands available to your project.

            ``` bash
            docker@gatsby:/var/www/gatsby$
            ```

            Remember, you can exit out of your service by running the `exit` command to get back to your host machine's command line.

{{% notice tip %}}
The completed code for Project 3 can be found at https://github.com/JDDoesDev/docksal-training-projects/tree/adv-cust-project-3
{{% /notice %}}
