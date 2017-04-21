# IawnCont
A primitive docker container management script for regenerating container from fresh images

Useful for setups not using docker-compose or more high level container orchestration

## About

It's a bash script that reads a convenient properties file that describes your container.
It then pulls the latest image and rebuilds your container using your supplied options.

## Why?

I kept on writing slightly different versions of the same script

Better to just have one script and an simple properties file.  (Better for me anyway)

Oh By the way, it's pronounced "yeown" and means "OK" in Welsh.  I'd rather not explain the full name in this context, suffice to say if you live in Caernarfon it is considered a warm greeting, elsewhere... not so much :)

## 1. Create a properties file to describe your container

Here's the bundled example which runs a shell loop.

    # File: bbtest.conf
    # the latest image tag you want to run
    VERSION=latest
    #
    # the image in full repository/image syntax if needed
    IMAGE=busybox
    #
    # the runtime name of your container (shouldn't change)
    CONT_NAME=bbtest
    #
    # bash array syntax for options for docker create
    CONT_OPTS=(-v "/tmp/rob:/tmp/rob")
    #
    # bash array syntax for extra docker CMD args
    CONT_ARGS=(sh -c "while true; do echo 'running'; sleep 10; done")
    #
    # link the docker host's IP address as host "docker.local" in the container
    LINK_LOCAL=false

## 2. Do the Re-Create dance

Calling the script with the path to the properties file will pull the image,
but will only build the container if it isn't running
If you want to do a lengthy pull before shutting down the container just run it first anyway.

1. edit /path/to/properties.conf and bump VERSION

2. `/path/to/recreateContainer /path/to/properties.conf` # to pull
3. Stop your container so it can be rebuilt (E.g. docker stop \<container\>)
4. `/path/to/recreateContainer /path/to/properties.conf` # to re-create
5. Start your new container. (E.g. docker start \<container\>)


To back-to-back the whole show, chain the routine as per the bundled example:

    /recreateContainer bbtest.conf; docker stop bbtest; ./recreateContainer bbtest.conf; docker start bbtest


## 3. Systemd

For my convenience I have bundled the systemd template from https://docs.docker.com/engine/admin/host_integration/
Ignore this for other process management regimes.


#### Initial Setup

1. `cp docker-container@.service /etc/systemd/system/`
2. `systemctl enable docker-container\@.service`
3. `./recreateContainer bbtest.conf`
4. `systemctl enable docker-container\@bbtest.service`

#### Re-Create and restart with the latest images

1. edit bbtest.conf and bump version
2. `./recreateContainer bbtest.conf` # to pull
3. `systemctl stop docker-container@bbtest.service`
4. `./recreateContainer bbtest.conf` # to recreate
5. `systemctl start docker-container@bbtest.service`
