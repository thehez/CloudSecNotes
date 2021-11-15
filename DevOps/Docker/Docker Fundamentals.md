
# Fundatmentals of Docker

- [Fundatmentals of Docker](#fundatmentals-of-docker)
  - [What is Docker?](#what-is-docker)
  - [Docker Architecture](#docker-architecture)
    - [Docker Images](#docker-images)
    - [Docker Registry](#docker-registry)
    - [Containers](#containers)
    - [Underlying technology](#underlying-technology)
  - [Useful Links and Further Information](#useful-links-and-further-information)

## What is Docker?

Docker is a developer tool that performs host OS virtualisation aka "containerisation". As these containers share the host OS they are more efficient than hypervisors/vms. 

Docker is used to create, run and deploy applications in containers. A Docker image contains the application's code, libraries, tools, dependencies and other files needed to make an application run. This solves the problem of "it works on my machine" by making it easy to package apps and their dependancies into one bundle - the container. 

Note - Due to the nature of containers being lightweight and portable they integrate well into Cloud and Continuous Integration/Continuous Deployment (CI/CD) environments. 

## Docker Architecture

Docker uses a client-server architecture in which the client communicates with the docker daemon to manage the building and running of containers. There are several components of the `docker engine`:

- Docker daemon (dockerd) - Listens for API calls to manage docker objects
- Docker client (docker) - The primary way users interact with docker, running docker commands sends the API calls to dockerd.
- Docker registry - Stores docker images - further described below... 
- Docker objects - When you use Docker, you are creating and using images, containers, networks, volumes, plugins, and other objects. The following section is a brief overview of some of those objects.

### Docker Images

Essentially a Docker image is a set of instructions for Docker to automatically build a container by reading instructions from a `Dockerfile`. A dockerfile is a text document similar to a template in that it contains all the commands a user would call on the command line to assemble an image. The `docker build` command docker will automatically build the docker file into an image. 

An example dockerfile:

```
FROM node:12-alpine
RUN apk add --no-cache python3 g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]

```

### Docker Registry 
Docker images can be built and stored locally using a dockerfile as described above, but most likely, images will be stored in a container registry. A container registry can be public, such as with [docker hub](https://hub.docker.com/) or private as seen with each cloud provider's internal offerings i.e. AWS has Elactic Container Registry. Images can be retrieved from the container registry by authenticating the docker client to the repository using the `docker login` command, and then pulling the image using the `docker pull` command.

Note - Best security practice dictates restricting the pulling of images from public repositories (as these could contain vulnerabilities/malware etc). Instead containers should be stored in a private repository that performs image scanning to identify security issues before containers are deployed.

### Containers

A container is an instance of an image at runtime. You can create, start, stop, move, or delete a container using the Docker API or CLI. You can connect a container to one or more networks, attach storage to it, or even create a new image based on its current state.

By default, a container is relatively well isolated from other containers and its host machine. You can control how isolated a container’s network, storage, or other underlying subsystems are from other containers or from the host machine.

A container is defined by its image as well as any configuration options you provide to it when you create or start it. When a container is removed, any changes to its state that are not stored in persistent storage disappear. Containers are predominately expected to run in the background, however its possible to create containers in 'interactive' mode and connect to them. 

### Underlying technology

- containerd is a container runtime which can manage a complete container lifecycle - from image transfer/storage to container execution, supervision and networking.
- container-shim handle headless containers, meaning once runc initializes the containers, it exits handing the containers over to the container-shim which acts as some middleman.
- runc is lightweight universal run time container, which abides by the OCI specification. runc is used by containerd for spawning and running containers according to OCI spec. It is also the repackaging of libcontainer.
- grpc used for communication between containerd and docker-engine.
- OCI maintains the OCI specification for runtime and images. The current docker versions support OCI image and runtime specs.


## Useful Links and Further Information
- [dockerlabs - docker commands cheatsheet ](https://dockerlabs.collabnix.com/docker/cheatsheet/)
- [docker overview - underlying technology ](https://docs.docker.com/get-started/overview/#the-underlying-technology)
- [docker blog - understanding the containerd runtime ](https://www.docker.com/blog/what-is-containerd-runtime/)
- [docker stackoverflow - how containerd compares to runc ](https://stackoverflow.com/questions/41645665/how-containerd-compares-to-runc)

