---
title : Docker buildx for multiarchitectural image building
date : 2023-06-23 02:54:00 +0900
categories : [Distributed Systems, Docker]
tags : [docker, cloudcomputing, kubernetes] #소문자만 가능
pinned : 0
---
# Multi-architectural image
First of all, what is a multi-architectural image?
Most developers disregard distinguishing CPU architectures as they are too 'low-level'.  
However with the <b>ARM64</b> quickly emerging into the industry, it is now necessary to take them into account for development.

## AMD64 and ARM64
As we know, Intel is known for its <b>x86 Architecture</b> while <b>AMD</b> is also a CPU that is compatible with the Intel CPU.  
However, <b>ARM</b> provides a completely different structure apart from the aforementioned two. It is usually used in smartphones for reasons of its reasonable price and low-power usage. Nowadays, it gradually is being used in more of the computer models, for example, AWS's new <b>graviton</b> instance or Apple's <b>M1</b>. That is why many of the mobile apps can be run natively on mackbooks with an ARM CPU.

So long story short, if you are not aware of the proper CPU type for your model, it simply will not be compatible for any of the apps you want to use.

## Emulators
Then how is it that Apple users are able to download and use the same applications as, say, PCs with AMD chips? Although most users are not even aware of it, Macbooks use <b>Rosetta</b> as an emulator which serves as a bridge between Intel and Apple. It adds a layer in between and enables a transition for software that requires different platforms.
With this extra layer, it adds a little overhead and may cause the performance to degrade(although it shouldn't be too noticeable).

# Buildx
So from Docker 19.03, it has slowly started to support multi-architectural builds, meaning that it allows for emulation. It enables creating cross-platform Docker images to run on a machine of your choice. It was announced in version <b>19.03</b> as an experimental component.

# Installation
## Brew install
For linux users, I find using homebrew to be the most effective way.

```bash
brew install docker-buildx
```

Once the installation is done, make sure to <b>link</b> buildx with docker.
After that confirm the installation with

```bash
docker buildx version
```

## Config.json
As an alternative way if you don't have homebrew on you machine, you can manually configure Docker to enable buildx.
```bash
export DOCKER_CLI_EXPERIMENTAL=enabled
vi ~/.docker/config.json 
```

In the json file, set the <b>experimental</b> field to `enabled`.

```json
"experimental": "enabled"
```