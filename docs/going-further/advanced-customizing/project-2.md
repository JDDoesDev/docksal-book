---
title: "Project 2: The Front-end Team Requires a Specific Version of NPM and NodeJS"
weight: 9
---

### Scenario

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

    3. Change the Dockerfile user so that settings aren't changed as `root`

        ``` dockerfile
        USER docker
        ```

    4. Run commands that will install and lock the version of NodeJS and NPM.

        ``` dockerfile
        RUN set -e; \ # Note the ';' and the '\'.
          # Initialize the user environment (this loads nvm)
          . $HOME/.profile; \
          # Install the necessary nodejs version and remove the unnecessary version
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
      ```

        Notice that every command in the `RUN` directive is followed with `; \` except the last line. This is because `RUN` directives are concatinated into a single line when run.

        We're sourcing the `~/.profile` file and getting our NVM (Node Version Manager) alias early on so that we can choose our version. In this case, we're using 8.11.0. Following that, we set this version as our default.

        Next, we install our locked version of NPM, clean up our caches, and then run some permission fixes, but we're not done yet.

    5. Return control to the `root` user.

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
