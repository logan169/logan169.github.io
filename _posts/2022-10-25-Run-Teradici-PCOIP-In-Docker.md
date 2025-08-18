---
title: 'Run Teradici PCOIP In Docker(Linux)'
tags: Docker Pcoip
---

With the pandemic and the shift to a remote working culture, we have been using Teradici Pcoip at my company for work.
Issues rised when the supported Pcoip company version wasn't compatible with latest ubuntu version. In order to be able
to keep working while using the latest unbuntu version, we can use docker to run an older ubuntu version that is compatible
with the supported company's version.

I've tried to look into Teradici official docker documentation, but this sadly hasn't been updated in a long time since 
the download portal they use in the Dokerfile is down since quiet some time now. I've been spending some times reading blogs to figure 
out how to workaround the existing issues it have and ended up with this code that should be put in a Dockerfile.


## Dockerfile
```
FROM ubuntu:22.04

# Use the following two lines to install the Teradici repository package
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get install -y curl
RUN curl -1sLf https://dl.anyware.hp.com/DeAdBCiUYInHcSTy/pcoip-client/cfg/setup/bash.deb.sh | distro=ubuntu codename=jammy bash

# Install apt-transport-https to support the client installation
RUN apt-get update && apt-get install -y apt-transport-https libx11-xcb1 libxcb1 libxcb-render0 libxcb-shape0 libxcb-xfixes0

# Install the client application
# here we precise the version we want, you can do a apt list -a pcoip-client to see available versions
# obviously the version needs to match with the ubuntu image version
RUN apt-get install -y pcoip-client


# Setup a functional user within the docker container with the same permissions as your local user.
# Replace 1000 with your user / group id
# Replace master with your local username
RUN export uid=1000 gid=1000 && \
    mkdir -p /etc/sudoers.d/ && \
    mkdir -p /home/master && \
    echo "master:x:${uid}:${gid}:master,,,:/home/master:/bin/bash" >> /etc/passwd && \
    echo "master:x:${uid}:" >> /etc/group && \
    echo "master ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/master && \
    chmod 0440 /etc/sudoers.d/master && \
    chown ${uid}:${gid} -R /home/master

# Set some environment variables for the current user
USER master
ENV HOME /home/master

# Set the path for QT to find the keyboard context
ENV QT_XKB_CONFIG_ROOT /user/share/X11/xkb

ENTRYPOINT exec pcoip-client
```

## Usage

### Create docker image
```
# from the Dockerfile file dir run
docker build . pcoip-client
```

### Create an alias
```
alias pcoip-client=='xhost +local:root && sudo docker run -d --rm -h myhost \
  -v $HOME/.config/:$HOME/.config/Teradici \
  -v $HOME/.logs:$HOME/.logs \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=$DISPLAY \
  -e TZ=America/New_York \
  pcoip-client'
```

### Launch Pcoip-Client
```
pcoip-client
```
