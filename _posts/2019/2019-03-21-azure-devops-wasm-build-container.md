---
layout: post
title: "Using Containers in Azure Devops to Build Mono Apps"
date: 2019-03-21 00:00:00 -0500
comments: true
category: Archive
comments: true
tags: [".NET", "devops", "azure", "mono"]
author: Jerome
---

The Azure Devops team recently added a new feature that gives the ability to run a build definition inside of a container pull from the Docker Hub. That container is running directly on the build host, executing the steps of the definition.

As I've been battling recently to get the Uno Platform Wasm-AOT Linux builds to run in a consistent context, without having to much to maintain, this feature comes in handy!

<!-- more -->
## How to build inside a container

The whole feature holds into two pieces of the yaml file definition.

First, the [resources that describes the container images](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema#container-resource) that can be used in the build definition:

```yaml
resources:
  containers:
  - container: nv-bionic-wasm
    image: nventive/wasm-build:1.0-bionic
```

The [`wasm-build` image here](https://github.com/nventive/docker/blob/master/wasm-build/Dockerfile) is built on top of the [.NET Core Bionic image](https://hub.docker.com/_/microsoft-dotnet-core-sdk) found on the docker hub, and contains all the Mono tools and Emscripten.

Then, to build something using this image, the [container reference](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema#container-reference) is added in a job definition:

```yaml
container: nv-bionic-wasm
```

And that's it !

The rest of the build definition will run inside the container, such as bash scripts.

In the case of Uno Platform based apps, this means that it's possible to build a complete Full-AOT or Mixed Interpreter/AOT Mno based app in complete isolation.

## Overall Build Time
The only drawback of this approach I could find for now is the setup time in the context of Azure hosted agents. The image's layers are not cached by the build host, and it takes about 3 minutes to initialize. Using a custom agent may be a faster to build using containers.

## About the default user for build scripts
Another detail to keep in mind is that the container is being setup by the build agent in a way that the build definitions scripts are executed under a non-root user.

Here's what the agent runs in the container right after starting it:

```bash
## Try create an user with UID '1001' inside the container.
/usr/bin/docker exec xxxx bash -c "grep 1001 /etc/passwd | cut -f1 -d:"
/usr/bin/docker exec xxxx useradd -m -u 1001 vsts_azpcontainer
## Grant user 'vsts_azpcontainer' SUDO privilege and allow it run any command without authentication.
/usr/bin/docker exec xxxx groupadd azure_pipelines_sudo
/usr/bin/docker exec xxxx usermod -a -G azure_pipelines_sudo vsts_azpcontainer
/usr/bin/docker exec xxxx su -c "echo '%azure_pipelines_sudo ALL=(ALL:ALL) NOPASSWD:ALL' >> /etc/sudoers"
```

This is certainly a best practice, but it prevents the use of the official distributions, where `sudo` is not installed.

I tried a bit to [work around this](https://github.com/Microsoft/azure-pipelines-agent/issues/2043), but in the end, the time needed to build the wasm-build image is about a hour on the docker hub. It makes sense to create an image that is properly configured to begin with.

## A sample definition using containers
The [Uno.Wasm.Bootstrapper](https://github.com/nventive/Uno.Wasm.Bootstrap) project is using new feature in [its build linux definition](https://github.com/nventive/Uno.Wasm.Bootstrap/blob/8c65fa35cd0641efbf4212521f852d43f268a3dc/.vsts-ci.yml#L55), which creates all the Full AOT and Mixed interpreter/AOT builds.