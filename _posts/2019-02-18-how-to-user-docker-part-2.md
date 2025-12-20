---
title: How to Use Docker (Part 2)
categories:
- DevEnv
- Shell
tags:
- docker
- virtual machine
- devop
- deployment
- container
- docker image
- docker hub
- dockerfile
- port
- volume
- docker compose
description: More introduction of docker, docker image, container, dockerfile
date: 2019-02-18
author_profile: true
classes: wide
---

# How to Use Docker (Part 2)

[How to Use Docker(Part 1)](https://www.notion.so/defd626f-321b-4531-9ce3-0333c35d45b9) introduced the very basic of docker usage, this part 2 will add more details and also `docker-compose`.

## Dockerfile

In part 1, we saw how to create a customized docker image with a very basic `Dockerfile`:

```bash
FROM ubuntu:xenial
LABEL maintainer="hello@example.org"
RUN apt-get update   
RUN apt-get install -y python3
```

Let's explore what we can also do with the docker file:

### Start the container with non-root user

Container will run with root user by default, but if we want to run the container not using the root user, we can specify user:

```bash
FROM centos:latest
RUN useradd -ms /bin/bash test_user # add a new user with root privilege
USER test_uesr # now using the new test_user
```

Note that once you specify the new user, the subsequent lines will be executed as *test_user* instead of root user. (after you run the image with `-it` , use `whoami` to check current user.

What if sometimes you want to go into the container as root user to do something? you can run `[docker exec](https://docs.docker.com/engine/reference/commandline/exec/)` with a specified user, for example, `docker exec -u 0 -it dreamy_kalam /bin/bash` will enable you to run commands as  rootuser for container `dreamy_kalam.`

### RUN , CMD, ENTRYPOINT

```bash
FROM ubuntu:xenial
RUN apt-get update
RUN apt-get install -y python3
CMD echo "hello world"
```

If you build and run the image with a simple `docker run [image id]`, you should see `hello world` printed in the output, so what's the difference between `RUN` and `CMD`? It looks like they are both executing a command, but `RUN` will commit the image changes for next step, it is happening during building, whereas `CMD` is the command the container executes by default when you launch the built image, and a docker file can only have one CMD. However, you can override the CMD if you pass some extra parameter when running the container, for instance:  `docker run [same image id as above] /bin/bash`, you won't see *hello world* because the it has been overridden by the `/bin/bash` .

`ENTRYPOINT` is similar to `CMD` to some extent, one of the differences is unlike *CMD*, *ENTRYPOINT* will not be ignored when docker container runs with command line parameters, using the same example:

```bash
FROM ubuntu:xenial
RUN apt-get update
ENTRYPOINT echo "this command is from entrypoint"
CMD echo "hello world"
```

Build and run `docker run [new image id] /bin/bash` you should see *this command is from entrypoint* is printed even though we passed `/bin/bash`.

### Set environment variables

You can use multiple lines or one line to specify environment variables:

```bash
ENV AWS_KEY 123
ENV AWS_SECRET abc

or
ENV AWS_KEY=123 AWS_SECRET=abc
```

Or you can set the env var with `RUN echo "EXPORT 123" >> /etc/fake.list`. Btw, you can specify it while running docker `docker run --env <key>=<value>`.

### Port Exposure

I mentioned briefly about port redirect with container like `docker run -p 8080:80 nginx` , if we check the official nginx dockerfile [here](https://github.com/nginxinc/docker-nginx/blob/master/mainline/alpine/Dockerfile#L142): you will see `EXPOSE 80`, this means the container listens on port 80 at runtime, but it does NOT make the port 80 of the container accessible to the host. At this point, the EXPOSE instruction exposes port 80 and makes it available only for inter-container communication (containers are running in the same docker network), if we do `docker inspect [container id]` find the IPAddress of the nginx container, then do `curl http://[nginx container Ip addres]:80` , we should see a nginx welcome page, but it won't work if we do `curl http://localhost:80`. To make port 80 available to the host, just run `docker run -p 8080:80 nginx` then you should be able to see the nginx welcome page with `curl http:localhost:8080`

You don't have to specify `8080` in above example: `docker run -d -p 80 nginx`, it will still expose port 80 but with a port specified randomly (within a range), run `docker ps` you should see something like this: `0.0.0.0:32769->80/tcp`, now you can use 32769 as the port on localhost.

If there are multiple ports available, you can use `docker run -d -P [an image with multiple ports]` it will automatically mapping the ports exposed in container with port 32769 and up.

### Images and Container Management

In previous examples, I created images with Dockerfile and directly build it with `docker build .`, those images don't have proper names, and it's better to give them a name and tag so we know which image it is, we can do this with `docker build -t myimage:v1 .`

When you run `docker ps`, you should see containers with random names, but we can give it meaningful names with `docker run --name mycontainername [image]`, or we can rename it with `docker rename [old container name] [new container name]`.

### Volume Management

What is volume in docker? It is one approach to [manage data in docker](https://docs.docker.com/storage/), by default all files created inside a container don't persist, it will be gone if the container no longer exists, and it's difficult to move the data else where if we need it. Docker has a few options to store files in the host machine instead of the writable container layer: volumes, bind mounts, or tmpfs mount if you are on linux.

The difference between these approaches is basically where to save data, for volumes, it is stored in part of the host filesystem which managed by Docker(`/var/lib/docker/volumes/` on Linux), it is the [preferred way](https://docs.docker.com/storage/volumes/) for persisting data. Bind mounts allow you to specify anywhere on the host system, tmpfs mounts are stored in the host system memory only.

Let's create a directory in a container then use the volume:

```bash
docker run -it --name mytest1 -v /mydata centos:latest /bin/bash
# we create a mydata directory
# we are now in the container
$ df -h
#you should see a /mydata directory, let's create a file here
$ cd /mydata
$ echo "test a file here" > mydata1.txt

# let's exit container and go back to the host
# go to the directory managed by Docker
# on linux it is /var/lib/docker/volumes/
$ cd /var/lib/docker/volumes/

# you should see some files here, but how do we know
# which one is mapping the /mydata directory?
docker inspect [the container we just started, mytest1]

# go to the Mounts section, and there is a Source key
# use that key to find the right volume in the volumes directory on host
# you should see the mydata1.txt we created earlier
# if you change the file in the host, the change will be reflected in the container

# if you already have a directory on host, you can re-map it with
# syntax similar to port mapping, such as
# docker run -it --name mytest2 -v /myhost/home/testDir:/mydata
```

## Docker Commands

### Inspect container process

There are a couple of ways to see what is going on in a container, includes:

- `docker top [container id]` is basically running top in your terminal but for the container
- `docker stats [container id]` is similar but will be updated periodically
- `docker network ls` will list network related info
- `docker event` to check events, `docker event —since '1h'` to check events for the past hour, or filter with `docker event —filter [type]`

## Docker Compose

[Docker compose](https://docs.docker.com/compose/overview/) can be used to define and run multiple container docker applications. The basic steps are:

1. Use Dockerfile to define the app's environment
2. Use `docker-compose.yml` to define services in the app, so that they can be run together in an isolated environment
3. Run `docker compose up` to start running the entire app, `docker compose down` to stop.

The [examples](https://docs.docker.com/compose/gettingstarted/) on docker is well written, if you understand how to write Dockerfile, the `docker-compose.yml` will look familiar to you with small syntax change, e.g lowercase instead of uppercase, I'll add more details and examples about docker compose later.
