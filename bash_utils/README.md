# bash_utils

## docker_dev

### Overview
`docker_dev` provides a development environment through Docker containers.

This is similar to `docker_bash`, but containers are persistent and re-entrant, so the development experience resembles working in a virtual machine, but without the overhead of virtualization (on Linux systems).

This script adds a base layer on top of the upstream docker images used by `docker_do`, adding some conveniences into the filesystem and allowing users to optionally define their own layers and docker arguments.


### Getting started

Follow directions for [setting up for Docker-based builds](https://flyzipline.atlassian.net/wiki/spaces/SW/pages/1526202373/An+Introduction+to+Writing+Flight+Software+at+Zipline#Setting-up-for-Docker-based-builds).

If running on a system with Nvidia graphics, ensure that Nvidia drivers are installed.
`$ apt install nvidia-drivers-<xxx>` should suffice.
If your system also has integrated graphics (our ThinkPads do), run `$ prime-select nvidia` to tell it to prefer your Nvidia graphics card.

For convenience, add the _bash_utils_ folder to your path in _.bashrc_:

```bash
export PATH="${PATH}:~/github/utils/bash_utils"
```

Now run `docker_dev`.
If you're using Nvidia graphics, you'll be prompted to allow installation of the Nvidia container runtime.
This will restart your docker daemon, so ensure you have no containers doing important work.

Once images are pulled and new layers are built, you should be dropped into a bash prompt within your container.

This will take a long time to run the first time, but once everything is cached, rebuilding and entering the container should be significantly faster.

### Working with docker_dev

You now have a persistent container running.
To enter the container, run `$ docker_dev` from any terminal (you may enter the container from multiple terminals at once, if you wish).
To leave the container, type `$ exit`.
To delete the container and blow away any changes to the container's filesystem, run `$ docker_dev delete`.

Changes to the container's filesystem will persist between sessions, but they don't affect your host filesystem.
This means you can `apt install` or `pip install` packages in your container as desired without changing your host system.
However, the _FlightSystems_ repository is mounted in by default, so any changes to this directory within the Docker container apply to your host filesystem.

Note that the build cache is also mounted in, so swithing between building in Docker and building in your host filesystem may poison your cache.
It's best to commit to just building within Docker or to mount a cache volume specific to Docker-based builds.

Occasionally, the upstream Docker image will contain important changes we need, such as updated toolchains and sysroots.
To pull in these changes and rebuild your development environment, run `$ docker_dev update`.

### Customizing docker_dev

The provided Docker image contains a minimal set of packages for building and running our targets.
You may want to add your own customizations to the image to pull in their preferred development tools.
This can be done by copying _docker_dev_files/dockerfile.user.example_ to _docker_dev_files/dockerfile.user_ and modifying as appropriate.
Changes to this file will apply to all future containers that `docker_dev` spins up.

You may also wish to mount in host directories or files (_.bashrc_ or a data directory, for example) or specify persistent environment variables that are not brought in by default.
This can be done by copying _docker_dev_files/user_config.json.example_ to _docker_dev_files/user_config.json_ and modifying as appropriate.

Changes to these files won't affect the container until you run `$ docker_dev delete` and spin up a new container.

### Using VS Code

VS Code provides an excellent containerized development experience through the `Dev Containers` extension.

To set up your system with a minimal `Dev Containers` configuration, run `$ docker_dev configure_code`.
This will install the `Dev Containers` extension and a configuration file specific to your system.

Ensure that a container is running by executing `$ docker_dev start`.

You should now be able to run VS Code, hit F1, and select `Dev Containers: Attach to Running Container`.
This will install the necessary extensions into your container and allow you to work in the containerized environment.

See [Full VSCode debug setup](https://flyzipline.atlassian.net/wiki/spaces/~712020aea05969e33d4dec8189900a4134040b/pages/3295412270/Full+VSCode+debug+setup+in+Docker) for very detailed instructions on setting up VS Code for containerized development.