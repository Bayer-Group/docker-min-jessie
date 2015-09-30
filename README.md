# docker-min-jessie

Updated minimally sized Debian Jessie build.

# Usage

If you just want to use this image, pull it and configure what you want:

    # docker pull monsantoco/min-jessie

To use this image in your own project, use this FROM line in your Dockerfile:

    FROM monsantoco/min-jessie:latest

# Building

Part of this process is documented in the Dockerfile, but the manual process is
outlined here.  Unfortunately, the Dockerfile approach is unable to actually
make the image smaller because it retains the parent images.  You need to
[docker-squash](https://github.com/jwilder/docker-squash) to actually make the
image smaller.  To our knowledge, there is no way to trick Docker Hub into
doing this for you as part of its automatic build.  So, you are stuck doing it
by hand.

This recipe for creating a minimal Debian GNU/Linux image is based on Phil
Cryer's recipe that can be found at
https://registry.hub.docker.com/u/philcryer/min-wheezy/

## Pull Original Image

First pull the official, larger Debian GNU/Linux image, in this case jessie.

    # docker pull debian:jessie

## Get Image ID

Run the command below get the image ID.

    # docker images
    REPOSITORY  TAG     IMAGE ID      CREATED     VIRTUAL SIZE
    debian      8.0     0e30e84e9513  3 days ago  122.8 MB
    debian      jessie  0e30e84e9513  3 days ago  122.8 MB
    debian      8       0e30e84e9513  3 days ago  122.8 MB

Look for the ID of the image you just pulled.  Above it would be
`0e30e84e9513`.

## Run Container

Run the container interactively using the command below.

    # docker run -it IMAGE_ID /bin/bash

## Clean

Within the interactive container, run the following command.

    # apt-get update && \
        apt-get upgrade -y && \
        apt-get clean -y && \
        apt-get autoclean -y && \
        apt-get autoremove -y && \
        rm -rf /usr/share/locale/* && \
        rm -rf /var/cache/debconf/*-old && \
        rm -rf /var/lib/apt/lists/* && \
        rm -rf /usr/share/doc/* && \
        exit

## Get Container ID

Run the command

    # docker ps -a
    CONTAINER ID  IMAGE     COMMAND      CREATED         STATUS                     PORTS  NAMES
    4d08ec826529  debian:8  "/bin/bash"  28 minutes ago  Exited (0) 47 seconds ago         berserk_sammet

and look for the ID of the container you were just running.  In this case, the
container ID is `4d08ec826529`.

## Commit Container

Run the command below to commit the container.  If you are doing this yourself,
you would replace `monsantoco` with your Docker Hub user or organization name.

    # docker commit CONTAINER_ID monsantoco/min-jessie:latest

This creates an image from the container, but it still has layers with all the
stuff we removed.

    # docker images
    REPOSITORY             TAG     IMAGE ID      CREATED         VIRTUAL SIZE
    monsantoco/min-jessie  latest  2319ddb6394e  58 seconds ago  122.8 MB
    debian                 8.0     0e30e84e9513  3 days ago      122.8 MB
    debian                 jessie  0e30e84e9513  3 days ago      122.8 MB
    debian                 8       0e30e84e9513  3 days ago      122.8 MB

It's the same size!  So we need to squash it.  Note the IMAGE_ID for your image
above.

## Squash Image

We squash the image using
[docker-squash](https://github.com/jwilder/docker-squash).  You will need to
install that tool for this step.  Use the IMAGE_ID for your image from the
previous step.

    # docker save IMAGE_ID | docker-squash -from root -t monsantoco/min-jessie | docker load

Now we can see that the image is smaller.

    # docker images
    REPOSITORY             TAG     IMAGE ID      CREATED         VIRTUAL SIZE
    monsantoco/min-jessie  latest  d9211ded36f0  54 seconds ago  82.46 MB
    debian                 8.0     0e30e84e9513  3 days ago      122.8 MB
    debian                 jessie  0e30e84e9513  3 days ago      122.8 MB
    debian                 8       0e30e84e9513  3 days ago      122.8 MB

# Copyright

Copyright (c) 2015, Monsanto Company

# License

This code is released under the modified BSD 3-clause license.  See [LICENSE](LICENSE)
file for details.
