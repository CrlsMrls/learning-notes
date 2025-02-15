# Docker

Docker can be used as a container runtime for Kubernetes. Although it is not the defaul container runtime for Kubernetes, it is still widely used.

Other container runtimes include containerd, CRI-O, and others.

## Docker Commands
This is a very basic list of the most basic commands for working with docker.

## 1. List docker images in the local machine

```bash
$ docker images
REPOSITORY                      TAG           IMAGE ID       CREATED        SIZE
redis                           latest        eca1379fe8b5   5 months ago   117MB
mysql                           latest        8189e588b0e8   5 months ago   564MB
nginx                           latest        6efc10a0510f   5 months ago   142MB
postgres                        latest        ceccf204404e   5 months ago   379MB
nginx                           alpine        8e75cbc5b25c   5 months ago   41MB
alpine                          latest        9ed4aefc74f6   5 months ago   7.04MB
ubuntu                          latest        08d22c0ceb15   6 months ago   77.8MB
nginx                           1.14-alpine   8a2fb25a19f5   4 years ago    16MB
```

## 2. List running containers

The `docker ps` command lists the running containers in the local machine.

```bash
$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
...
```

## 3. Build a docker image

Build a docker image using the Dockerfile and name it `webapp` with tag `slim`:
```bash
$ docker build -t webapp:slim . 
```

## 4. Run a docker image

Run the docker image `webapp` and map the port 8080 of the container to the port 8282 of the host:
```bash
$ docker run -p 8282:8080 webapp
```

## 5. Stop a running container

Stop a running container using the `docker stop` command:
```bash
$ docker stop <container_id>
```

## 6. list useful information about a container

To check the base image of a docker image:

```bash
$ docker run python:3.6 cat /etc/*release*
PRETTY_NAME="Debian GNU/Linux 11 (bullseye)"
NAME="Debian GNU/Linux"
VERSION_ID="11"
VERSION="11 (bullseye)"
VERSION_CODENAME=bullseye
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```


