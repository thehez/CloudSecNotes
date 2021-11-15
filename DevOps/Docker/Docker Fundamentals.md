
# Fundamentals of Docker

- [Fundamentals of Docker](#fundamentals-of-docker)
  - [What is Docker?](#what-is-docker)
  - [Docker Architecture](#docker-architecture)
    - [Docker Images](#docker-images)
    - [Docker Registry](#docker-registry)
    - [Containers](#containers)
  - [Other container technology](#other-container-technology)
    - [Control Groups](#control-groups)
    - [Kernel Namespaces](#kernel-namespaces)
  - [References, Links and Further Information](#references-links-and-further-information)

## What is Docker?

Docker is a developer tool that performs host OS virtualisation aka "containerisation". As these containers share the host OS they are more efficient than hypervisors/vms. 

Docker is used to create, run and deploy applications in containers. A Docker image contains the application's code, libraries, tools, dependencies and other files needed to make an application run. This solves the problem of "it works on my machine" by making it easy to package apps and their dependancies into one bundle - the container. 

Note - Due to the nature of containers being lightweight and portable they integrate well into Cloud and Continuous Integration/Continuous Deployment (CI/CD) environments through the use of container orchestration platforms which manage the scheduling of containers - the most popular being kubernetes. 

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

## Other container technology

Docker is not the only container technology though it is the most popular, however recently kubernetes (k8s) has dropped docker in favour of containerd. The Open Container Initiative (and container runtime interface in the case of K8s) provides a set of standards and most of these tools work together. From a top down approach:

Image from Tutorialworks (see references):
![Container technology flow](https://www.tutorialworks.com/assets/images/container-ecosystem.drawio.png?raw=true "Title")

- Docker engine is what end users interact with to run docker commands 

- Containerd is a high-level container runtime that came from Docker, and implements the CRI spec. It pulls images from registries, manages them and then hands over to a lower-level runtime, which actually creates and runs the container processes. Containerd was separated out of the Docker project, to make Docker more modular, but Docker still uses containerd internally itself - when you install Docker, it will also install containerd.

- containerd also implements the Kubernetes Container Runtime Interface (CRI), via its cri plugin.

- container-shim handle headless containers, meaning once runc initializes the containers, it exits handing the containers over to the container-shim which acts as some middleman.
  
- runc is lightweight universal run time container, which abides by the Open Container Initiative (OCI) specification. runc is used by containerd for spawning and running containers according to OCI spec. It is also the repackaging of libcontainer.
  
- grpc is used for communication between containerd and docker-engine.
  
- OCI maintains the OCI specification for runtime and images. The current docker versions support OCI image and runtime specs.

Container ecosystem:
![Container ecosystem](https://containerd.io/img/architecture.png?raw=true "Title")

### Control Groups

cgroups are a feature of the linux kernel, that allow users to limit the access that processes and containers have to system resources such as CPU, RAM, IOPS and the network. 

Docker uses cgroups to enforce constraints on docker containers to prevent a single container from consuming all resources on the host i.e. to prevent forkbombs etc. This is particularly important in multi-tenant platforms.

### Kernel Namespaces

Namespaces are a feature of the linux kernel that provide isolation for fundamental aspects of containers from the host OS. Namespaces provide the first and most straightforward form of isolation: processes running within a container cannot see processes running in another container, or on the host system.

By default each container also gets its own network stack, meaning that a container doesn’t get privileged access to the sockets or interfaces of another container. From a network architecture point of view, all containers on a given Docker host are sitting on bridge interfaces. This means that they are just like physical machines connected through a common Ethernet switch.

Docker namespaces:
- PID - process isolation
- NET - manages network interfaces
- IPC - access to IPC resources
- MNT - filesystem mount points
- UTS - isolates kernel and version identifiers
- UID - UserID for privilege isolation

Security Note on UID, by default a root user in a docker container has the same priviges as the root user of the host - i.e. if a mount point has been connected to the docker container the conatiner root user can access that mount on the host OS. See docker security and hardening.

## References, Links and Further Information
- [dockerlabs - docker commands cheatsheet ](https://dockerlabs.collabnix.com/docker/cheatsheet/)
- [docker overview - underlying technology ](https://docs.docker.com/get-started/overview/#the-underlying-technology)
- [docker blog - understanding the containerd runtime ](https://www.docker.com/blog/what-is-containerd-runtime/)
- [docker stackoverflow - how containerd compares to runc ](https://stackoverflow.com/questions/41645665/how-containerd-compares-to-runc)
- [tutorialworks - understanding containerd runc oci etc... ](https://www.tutorialworks.com/difference-docker-containerd-runc-crio-oci/)
