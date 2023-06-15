> we need to go deeper

# Index

1. [Base](./README.md#Base)
   1. [Official Documentation](./README.md#Official-Documentation)
   2. [What's docker](./README.md#Whats-docker)
   3. [How to install](./README.md#How-to-install)
   4. [Why Docker != VM && Docker > VM](./README.md#Why-Docker--VM--Docker--VM)
   5. [A word about shell](./README.md#A-word-about-shell)

2. [Level of inception](./README.md#Level-of-inception) (**where the fun begins**)
   1. [Level 3: Dockerfile](./README.md#Level-3-Dockerfile)
   2. [Level 4: entrypoint](./README.md#Level-4-entrypoint)
   3. [Level 2: docker-compose.yaml](./README.md#Level-2-docker-compose.yaml)
   4. [Level 1: Makefile](./README.md#Level-1-Makefile)

3. [Final word](./README.md#Final-Word)

# Base

## Official documentation

docker / compose have a very good documentation, check out some link:

- [Dockerfile](https://docs.docker.com/engine/reference/builder/)
  - [`docker` (CLI)](https://docs.docker.com/engine/reference/commandline/cli/)
- [Compose](https://docs.docker.com/compose/compose-file/)
  - [Compose V3](https://docs.docker.com/compose/compose-file/compose-file-v3/)
  - [`compose` (CLI)](https://docs.docker.com/compose/reference/)

## What's docker

https://www.youtube.com/watch?v=-YnMr1lj4Z8
> a very good video on how docker works

TL;DR

Docker is client interface to use containerd.

Containerd is a service that use runc.

Runc is here to manage those little container with the unshare function.

Unshare is a system call that allow to have 'spaces' where an user can be root,
but stay a regular user outside of that spaces.

## How to install

- [installing docker engine](https://docs.docker.com/engine/install/)
  - [Debian](https://docs.docker.com/engine/install/debian)
  - [Ubuntu](https://docs.docker.com/engine/install/ubuntu)
- [installing docker rootless](https://docs.docker.com/engine/security/rootless/)
> installing in rootless means that docker will not have root privileges on host
> more secure in prod

### Why Docker != VM && Docker > VM

```bash
# If you we launch a shell on on docker images
docker run -it alpine ash

# We then check for all the PID for the current terminal, with the 'T' option
# in the container and then on the host
ps aT
```

One thing to notice is that the PID of the shell of the container appear in the
host ps output.

It's possible thanks to unshare, in the same manner where container have his own
spaces for the PID, the container also have his own spaces for the user.

That's allow, the root user in his spaces / container, and so many other cool thing :smirk:

> undocumented spaces

```bash
# communicate with the docker socket
curl --unix-socket /var/run/docker.sock http://localhost/images/json | jq
curl --unix-socket /var/run/docker.sock http://localhost/images|containers/[image_name/]json | jq
curl --no-buffer --unix-socket /var/run/docker.sock http://foo/events | jq

# current shell process, namespaces info
ls -la /proc/$$/ns
```

## A word about shell

Shell can be found on every layer in this project, learning some basic stuff
about bash could be very helpfull.

it can be found:
- LVL1 Makefile: those are shell command launched with dependencies rules
- LVL2 docker-compose.yaml: this is one is more subtil, you specify a bash
  variable with bracket (${}) for assigning value, this allow brace expension,
  mainly for default in the .env file
- LVL3 Dockerfile: when a `RUN` inst is evaluated, this a `/bin/sh -c` by default
- LVL4 entrypoint: this one is are pure shell script

a little introduction can be found [here](./rsc/intro_bash)

# Level of inception

## Level 3: Dockerfile

Docker images have two stage, the first one is the build stage, where you build
a good base for the run stage.

In the build stage you can:
- Choose the base image
- Install package you need
- copy file from host to container

But you cannot:
- start services
- create directory with RUN (you can use the WORKDIR instruction instead)

The build stage is describe in a **Dockerfile** with instruction.

But keep Dockerfile clean, as the entrypoint and a docker-compose.yaml are here to
support you.

---

[Dockerfile](./rsc/3_Dockerfile/Dockerfile)

Usefull `docker` commmand

```bash
# pull the base image with the specified tag
docker pull image_name:tag

# build images
docker build [-t image_name -f dockerfile_path] path

# launch an interactive (-it);
# exposing port (-p) hostp on the host, from contp in the container;
# making volumes (-v) from ./host/path to /container/path available
# from the image_name previously build, with the arguments provided
docker run -it -p hostp:contp -v ./host/path:/container/path image_name arguments

# view all running docker
docker ps

# remove almost all networks, volumes and images in one command
docker system prune -af

# list all images
docker images
```

## Level 4: Entrypoint

After the build stage we have the run stage, this is right after the container
have launched the entrypoint.

Having a script instead of a program help you to configure, or not, the
services, just before the main application / service launch.

Sometimes entrypoint are mandatory because configuration need to be perfomed on
a running services, ex: MariaDB.

We have to do 3 step in the [Dockerfile](./rsc/4_Entrypoint/Dockerfile) to make it work with the entrypoint:
1. `COPY` the the entrypoint into the container
2. set the `ENTRYPOINT` where the entrypoint script was copied on the container
3. set the default args (with `CMD`) for the `ENTYPOINT`

---

[entrypoint](./rsc/4_Entrypoint/entrypoint)

## Level 2: docker-compose.yaml

docker-compose.yaml or yml, are yaml that help to orchestrate all the Dockerfile
with compose you can easily manage shared volumes, network and many other thing.

Compose file are yaml so yaml rules apply, like 2 space for tabs etc..

docker-compose.yaml should start with the `version` tag, they are
3 other usefull tag:

- `services` all your "images"
- `networks` network configuration related
- `volumes` configure how the named volumes behave


We keep our from [python_server](./rsc/2_Compose/python_server) folder from the previous steps and we add an
other [Dockerfile](./rsc/2_Compose/python_client/Dockerfile) and his [entrypoint](./rsc/2_Compose/python_client/entrypoint) to simulate a client that will call
our server every 10 second.

It's important to note that we reference 'python_server' as the host in the
python_client entrypoint, this is possible thanks to docker compose seting up
the hostname as the services name specified in the docker-compose.yaml

> the [.env](./rsc/2_Compose/.env) file has to be in the same directory as the docker-compose.yaml
> or it should be specified with the `env_file` tag in the services

---

[docker-compose.yaml](./rsc/2_Compose/docker-compose.yaml)

Usefull `docker compose` commmand

```bash
# launch all the services in the docker-compose.yaml
docker compose up

# specify the docker-compose.yaml file (-f)
docker compose -f ./files/docker-compose.yaml up

# build the stack, usefull to not rebuild all the steps
docker compose build service_name

# run with a specific argument for the entrypoint
docker compose run -it service_name command ...

# execute command when service_name is running
docker compose exec -it service_name command ...

# view all running services
docker compose ps
```

## Level 1: Makefile

The Makefile is an extra layer to smooth last details, like running an entire
project with an only `make` is always satisfing.

it could help to create your shared directory needed by your compose volumes, or
even download file before the Dockerfile interact with it

---

[Makefile](./rsc/1_Makefile/Makefile)

# Final word

I named the second part with level because i think, that this project as to be
seen like this:
Makefile (1) that control docker-compose.yaml (2) that manage Dockerfile (3)
that launch entrypoint (4)

Sorry for the confusion that could lead, but i think that is important to see
this in this point of view

I hope you enjoyer, feel free to open issues / pull request to correct point or
just my grammar.
