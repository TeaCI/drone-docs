+++
date = "2015-12-05T16:00:21-08:00"
draft = false
title = "Quick Start"
weight = 1
menu = "usage"
toc = true
+++

# Overview

Tea CI is a continuous integration service for Windows developers, free for open source projects. Tea CI runs a [fork of Drone CI](https://github.com/TeaCI/drone) with additional support of Cygwin/Msys2 on Wine in order to compile Windows software. Any documentation for Drone CI should be a good reference for Tea CI. This documentation might mix the keyword Drone CI and Tea CI.

Before configure your build you need to connect your github repository to Tea CI. Firstly [login](https://tea-ci.org/login) to Tea CI with your github account, then search for your repository with the top right bar on Tea CI web UI, click your repository name and jump to `https://tea-ci.org/<user_name>/<repo_name>`, finally click the `ACTIVATE NOW` button.

In order to configure your build you must include a `.drone.yml` file in the root of your repository. This section provides a brief overview of the configuration file and build process.

Example .drone.yml configuration:

```yaml
---
# Build configuration for https://www.tea-ci.org

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

Once activated, every commit and pull request **automatically** send a hook from your version control system (ie GitHub) to Tea CI. These hooks instruct Tea CI to execute a new build.

When Tea CI receives a hook it fetches the `.drone.yml` from your repository and uses this as a blueprint for build execution. For the purposes of this tutorial let's assume your repository uses the following configuration:

```yaml
---
# Build configuration for https://www.tea-ci.org

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

Hooks specify commit details including branch and commit hash. Tea CI will **automatically** clone and checkout the commit into the build workspace:

```
$ git init
Initialized empty Git repository in /drone/src/github.com/TeaCI/xz/.git/
$ git remote add origin https://github.com/TeaCI/xz.git
$ git fetch --no-tags --depth=50 origin +refs/heads/master:
From https://github.com/TeaCI/xz
 * branch            master     -> FETCH_HEAD
 * [new branch]      master     -> origin/master
$ git reset --hard -q b3bd814618d82341dd22f2f59da3a1b467724eb0
```

# Images

Tea CI executes your build inside an ephemeral Docker image. This means you don't have to setup or install any repository dependencies on your host machine. Use any valid Docker image in any Docker registry as your build environment.

Example .drone.yml configuration uses the Tea CI official msys32 image:

```yaml
---
# Build configuration for https://www.tea-ci.org

build:
  image: teaci/msys32
```

The Tea CI project maintains several official docker images for our users. Currently we support two images: `teaci/msys64` and `teaci/msys32`. Cygwin support is ongoing.

# Shell

Tea CI executes your build using a custom shell you specified.

Example .drone.yml configuration uses the Tea CI official msys32 image using mingw32 shell:

```yaml
---
# Build configuration for https://www.tea-ci.org

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

`image: teaci/cygwin64` with `shell: cygwin64`, for target `x86_64-pc-cygwin`.

Note: while mingw32 shell and msys32 shell share the same image, usual Win32 application developers might only need the mingw32 shell to build "native" programs. The msys32 shell is mostly for Msys2 package maintainers. The cygwin32 shell is mostly for Cygwin package maintainers.

# Pull

Use the `pull` attribute to instruct Tea CI to always pull the latest Docker image. This helps ensure you are always testing your code against the latest image. We recommend you always using `pull: true` with our official Docker image like teaci/msys32 and teaci/cygwin32.

```yaml
---
# Build configuration for https://www.tea-ci.org

build:
  image: teaci/msys32
  pull: true
  shell: mingw32
```

# Commands

Tea CI previously cloned your source code into the build workspace. The build workspace is mounted into your Docker container at runtime as a volume. This means your code is cloned **outside** of the build container but your build commands are run **inside** of the build container.

Tea CI executes the following bash commands inside your build container:

```yaml
---
# Build configuration for https://www.tea-ci.org

build:
  image: teaci/msys32
  pull: true
  shell: mingw32
  commands:
    - ./configure
    - make
    - make check
```

# Cmd

Tea CI does not support Windows cmd shell directly, however Msys2 has a cmd wrapper. Use `cmd /c "command arg1 arg2"` in Msys2 shell:

```yaml
---
# Build configuration for https://www.tea-ci.org

build:
  image: teaci/msys32
  shell: mingw32
  commands:
    - cmd /c dir
    - cmd /c "echo %WINDIR%"
    - ./configure
    - make
    - make check
```

# Native Linux Shell

Tea CI supports Linux build as well, just set `shell: sh` in .drone.yml:

```yaml
---
# Build configuration for https://www.tea-ci.org

build:
  image: teaci/msys32
  shell: sh
  commands:
    - ./configure
    - make
    - make check
```

You can also try to play with matrix build in order to build both Linux binary and Windows binary in Tea CI.


# Build dependencies

`teaci/msys32` and `teaci/msys64` images has [Msys2](https://msys2.github.io) pre-installed, you can install [thousands of open source libraries](https://mirrors.tea-ci.org/msys2/mingw/) using `pacman`.

```yaml
---
# Build configuration for https://www.tea-ci.org
# Tea CI is a fork of Drone CI with Cygwin/Msys2 support
# Feel free to share Tea CI to more open source developers
# http://docs.tea-ci.org/usage/overview/
# Please add your project to https://github.com/TeaCI/tea-ci/wiki/Msys2-on-Wine#use-msys2-in-tea-ci

build:
  image: teaci/msys32
  pull: true
  shell: mingw32
  commands:
    - pacman -S --needed --noconfirm --noprogressbar mingw-w64-i686-libpng
    - ./configure
    - make
    - make check
```

If you are using `shell: sh` for Linux build, then you need `apt-get` rather then `pacman` for installing dependencies because teaci/msys32 is based on Ubuntu docker image, which might bring some inconvenience for matrix build configuration, but anyway it is doable. If you do not like teaci/msys32, you can also use any other image from https://hub.docker.com/explore/ or build your own image. However, matrix support is incomplete until Tea CI upgrades to Drone 0.5: http://readme.drone.io/0.5/usage/matrix/, as a result currently there is no way to control `image` and `shell` as a group in matrix build.

# Matrix build

You can use matrix build to compile both 32 bit binary and 64 bit binary at the same time. Refer [the documenation](http://docs.tea-ci.org/usage/matrix/) for more details.

```yaml
---
# Build configuration for https://www.tea-ci.org
# Tea CI is a fork of Drone CI with Cygwin/Msys2 support
# Feel free to share Tea CI to more open source developers
# http://docs.tea-ci.org/usage/overview/
# Please add your project to https://github.com/TeaCI/tea-ci/wiki/Msys2-on-Wine#use-msys2-in-tea-ci

build:
  image: teaci/msys$$arch
  pull: true
  shell: mingw$$arch
  commands:
    - if [ $$arch = 32 ]; then target=i686; fi
    - if [ $$arch = 64 ]; then target=x86_64; fi
    - pacman -S --needed --noconfirm --noprogressbar mingw-w64-${target}-pkg-config
    - ./autogen.sh
    - ./configure
    - make
    - make check

matrix:
  arch:
    - 64
    - 32
```

# Playground

Fork our [example code](https://github.com/teaci/xz) to get a quick start on Tea CI. Send pull request to our [playground](https://github.com/teaci/xz/pulls) to see how a pull request [trigger a Tea CI build](https://github.com/TeaCI/xz/pull/1). You are also welcome to contribute more demo repositories.

[![Build Status](https://tea-ci.org/api/badges/TeaCI/xz/status.svg?branch=master)](https://tea-ci.org/TeaCI/xz)

<!--
# Services

Tea CI supports launching separate, ephemeral Docker containers as part of the build process. This is useful, for example, if you require a database for running your unit tests.

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

Tea CI supports a large number of publish and deployment capabilities through external plugins. Plugins are Docker containers that are automatically downloaded, attach to your build, and execute a very specific publish or deployment task.

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

First Tea CI runs your build commands inside the teaci/msys32 container:

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

Tea CI executes publish and deployment plugins upon successful completion of the build step. Plugins are executed in separate Docker containers but have access to your build workspace. This means any files created and stored in the `/drone` workspace are available to plugins.

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

Download the [command line tools](/devs/cli) to build and test your code locally inside a Docker environment using the exact same build process as Tea CI. You should think of your `.drone.yml` file as a `docker-compose.yml` alternative that is optimized for repeatable, local testing.

Command to execute a local build from the command line:

```
drone exec
```
-->

# Getting Help

Please use the community [chat room](https://gitter.im/TeaCI/drone) for support.
Create an issue on [github repo](https://github.com/TeaCI/tea-ci/issues) if you found any bugs. If you need to extent your build time limitation, please drop a private message on gitter to @fracting.
