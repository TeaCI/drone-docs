+++
date = "2015-12-05T16:00:21-08:00"
draft = false
title = "Quick Start"
weight = 1
menu = "usage"
toc = true
+++

# Overview

Before configure your build you need to connect your github repository to Tea CI. Firstly [login](https://tea-ci.org/login) to Tea CI with your github account, then search for your repository with the top right bar on Tea CI web UI, click your repository name and jump to `https://tea-ci.org/<user_name>/<repo_name>`, finally click the `ACTIVATE NOW` button.

In order to configure your build you must include a `.drone.yml` file in the root of your repository. This section provides a brief overview of the configuration file and build process.

Example .drone.yml configuration:

```yaml
---
build:
  image: teaci/msys32
  shell: mingw32
  pull: true
  commands:
    - ./configure
    - make
    - make check
```

# Hooks

Once activated, every commit and pull request automatically send a hook from your version control system (ie GitHub) to Drone. These hooks instruct Drone to execute a new build.

When Drone receives a hook it fetches the `.drone.yml` from your repository and uses this as a blueprint for build execution. For the purposes of this tutorial let's assume your repository uses the following configuration:

```yaml
---
build:
  image: teaci/msys32
  shell: mingw32
  pull: true
  commands:
    - ./configure
    - make
    - make check
```

# Cloning

Hooks specify commit details including branch and commit hash. Drone will automatically clone and checkout the commit into the build workspace:

```
git clone --depth=50 --recusive=true \
    https://github.com/octocat/hello-world.git \
    /drone/src/github.com/octocat/hello-world

git checkout 7fd1a60
```

# Images

Drone executes your build inside an ephemeral Docker image. This means you don't have to setup or install any repository dependencies on your host machine. Use any valid Docker image in any Docker registry as your build environment.

Example .drone.yml configuration uses the Tea CI official msys32 image:

```yaml
---
build:
  image: teaci/msys32
```

The Tea CI project maintains several official docker images for our users. Currently we support two images: `teaci/cygwin32` and `teaci/msys32`. 64 bit support is ongoing.

# Shell

Drone executes your build using a custom shell you specified.

Example .drone.yml configuration uses the Tea CI official msys32 image using mingw32 shell:

```yaml
---
build:
  image: teaci/msys32
  shell: mingw32
```

We currently support the following combinations of images and shells:

`image: teaci/msys32` with `shell: mingw32`, for target `i686-w64-mingw32`.

`image: teaci/msys64` with `shell: mingw64`, for target `x86_64-w64-mingw32`.

`image: teaci/msys32` with `shell: msys32`, for target `i686-pc-msys`.

`image: teaci/msys64` with `shell: msys64`, for target `x86_64-pc-msys`.

`image: teaci/cygwin32` with `shell: cygwin32`, for target `i686-pc-cygwin`.

Note: while mingw32 shell and msys32 shell share the same image, usual Win32 application developers might only need the mingw32 shell to build "native" programs. The msys32 shell is mostly for msys2 package maintainers. The cygwin32 shell is mostly for cygwin package maintainers.

# Pull

Use the `pull` attribute to instruct Drone to always pull the latest Docker image. This helps ensure you are always testing your code against the latest image. We recommend you always using `pull: true` with our official Docker image like teaci/msys32 and teaci/cygwin32.

```yaml
---
build:
  image: teaci/msys32
  shell: mingw32
  pull: true
```

# Commands

Drone previously cloned your source code into the build workspace. The build workspace is mounted into your Docker container at runtime as a volume. This means your code is cloned **outside** of the build container but your build commands are run **inside** of the build container.

Drone executes the following bash commands inside your build container:

```yaml
---
build:
  image: teaci/msys32
  shell: mingw32
  pull: true
  commands:
    - ./configure
    - make
    - make check
```

<!--
# Services

Drone supports launching separate, ephemeral Docker containers as part of the build process. This is useful, for example, if you require a database for running your unit tests.

Example .drone.yml configuration with a Postgres database:

```yaml
---
build:
  image: teaci/msys32
  shell: mingw32
  pull: true
  commands:
    - ./configure
    - make
    - make check

-->

<!--
compose:
  database:
    image: postgres
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=mysecretpassword
```
-->

<!--
# Deployments

Drone supports a large number of publish and deployment capabilities through external plugins. Plugins are Docker containers that are automatically downloaded, attach to your build, and execute a very specific publish or deployment task.

Example .drone.yml configuration with the Docker publish plugin:

```yaml
---
build:
  image: teaci/msys32
  shell: mingw32
  pull: true
  commands:
    - ./configure
    - make
    - make check

publish:
  docker:
    username: octocat
    password: password
    email: octocat@github.com
    repo: octocat/hello-world
```

First Drone runs your build commands inside the teaci/msys32 container:

```yaml
---
build:
  image: teaci/msys32
  shell: mingw32
  pull: true
  commands:
    - ./configure
    - make
    - make check
```

Drone executes publish and deployment plugins upon successful completion of the build step. Plugins are executed in separate Docker containers but have access to your build workspace. This means any files created and stored in the `/drone` workspace are available to plugins.

The Docker plugin in our example runs `docker build` and `docker publish` after the build step successfully completes using the configuration parameters in the .drone.yml file:

```yaml
---
publish:
  docker:
    username: octocat
    password: password
    email: octocat@github.com
    repo: octocat/hello-world
```
-->

<!--
# Local Testing

Download the [command line tools](/devs/cli) to build and test your code locally inside a Docker environment using the exact same build process as Drone. You should think of your `.drone.yml` file as a `docker-compose.yml` alternative that is optimized for repeatable, local testing.

Command to execute a local build from the command line:

```
drone exec
```
-->

# Getting Help

Please use the community [chat room](https://gitter.im/TeaCI/drone) for support.
