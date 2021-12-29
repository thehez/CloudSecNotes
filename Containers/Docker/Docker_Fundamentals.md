
# Fundamentals of Docker

- [Fundamentals of Docker](#fundamentals-of-docker)
  - [What is Docker?](#what-is-docker)
  - [What is a Container?](#what-is-a-container)
  - [Docker Architecture](#docker-architecture)
    - [Docker Images](#docker-images)
    - [Docker Registry](#docker-registry)
    - [Containers](#containers)
  - [Other container technology](#other-container-technology)
  - [References, Links and Further Information](#references-links-and-further-information)

## What is Docker?
Docker is the most popular container developer tool which provides an easy to use interface for building, running, and managing containers on servers and the cloud. The term "docker" may refer to either the tools (the commands and a daemon) or to the Docker image - Dockerfile - file format. 

A Docker image contains the application's code, libraries, tools, dependencies and other files needed to make an application run. This solves the problem of "it works on my machine" by making it easy to package apps and their dependancies into one bundle - the container.

## What is a Container?
Containers are a form of virtualisation, but unlike virtual machines they share the hosts Operating System Kernel making them a lightweight portable alternative. Each ontainer has its own filesystem, share of CPU, memory, process space, and more. As they are decoupled from the underlying infrastructure, they are portable across clouds and OS distributions.

Note - Due to the nature of containers being lightweight and portable they integrate well into Cloud and Continuous Integration/Continuous Deployment (CI/CD) environments through the use of container orchestration platforms which manage the scheduling of containers - the most popular being kubernetes.

## Docker Architecture
The Docker engine consists of a client-server architecture in which the client communicates with the docker daemon to manage the building and running of containers. There are several components of the `docker engine`:

- Docker daemon (dockerd) - The server process that listens for API calls to process and manage docker objects
- Docker client (docker) - The primary way users interact with docker, running docker cli commands sends the API calls to dockerd.
- Docker registry - Stores docker images - further described below... 
- Docker objects - When you use Docker, you are creating and using images, containers, networks, volumes, plugins, and other objects. 

The following section is a brief overview of some of those objects.

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

Note - Best security practice dictates restricting the pulling of images from public registry (as these could contain vulnerabilities/malware etc). Instead container images should be stored in a private registry that performs image scanning to identify security issues before containers are deployed.

### Containers
A container is an instance of an image at runtime. You can create, start, stop, move, or delete a container using the Docker API or CLI. You can connect a container to one or more networks, attach storage to it, or even create a new image based on its current state.

By default, a container is relatively well isolated from other containers and its host machine. You can control how isolated a container’s network, storage, or other underlying subsystems are from other containers or from the host machine.

A container is defined by its image as well as any configuration options you provide to it when you create or start it. When a container is removed, any changes to its state that are not stored in persistent storage disappear. Containers are predominately expected to run in the background, however its possible to create containers in 'interactive' mode and connect to them. 

## Other container technology
Docker is not the only container technology though it is the most popular, though recently kubernetes (k8s) has dropped docker in favour of containerd. The Open Container Initiative (and container runtime interface in the case of K8s) provides a set of standards and most of these tools work together. From a top down approach:

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

## Docker Advantages & Use Cases
Docker provides application isolation with little overhead and a low memory footprint. Additionally Docker provides developers with a layer of abstraction by packaging code and dependancies together. As such it is possible to run many multiple containers on a single host which can be rapidly spun up and down in seconds which isolating dependancies from other potential overlapping depandancy requirements.

Common Use cases include:
- Configuration Management - Docker simplifies configurations; providing the capability of pooling an environment with a configuration into code, packaging and deploying it on any platform.

- Code Pipeline Management - Docker provides a consistent environment which eases the development and deployment process as code moves between various development, testing and production-like environments which may vary in infrastructure and configuration etc.

- Application Isolation - With multiple microservies serving an application it is likely that such services will require common libraries and packages which may differ in version requirements causing depenancy conflicts. Docker isolates microservices within their own environment with their own depenancies and configurations for seamless deployment.

- Continuous Intergration and Deployment (CI/CD) - As Docker provides image versionining it is possible to pull new code, build it, package it into a Docker image and push a new updated image to a repository. This can then be pulled via a CD tool to each development environment either on code push or frequancy.

- Consistent Environments -  Docker helps prevent the 'It works on my machine' situation by setting consistent environment variables and configuration settings in the image file, without any other variables that can affect the running of an application or service.

## References, Links and Further Information
- [dockerlabs - docker commands cheatsheet ](https://dockerlabs.collabnix.com/docker/cheatsheet/)
- [docker overview - underlying technology ](https://docs.docker.com/get-started/overview/#the-underlying-technology)
- [docker blog - understanding the containerd runtime ](https://www.docker.com/blog/what-is-containerd-runtime/)
- [docker stackoverflow - how containerd compares to runc ](https://stackoverflow.com/questions/41645665/how-containerd-compares-to-runc)
- [tutorialworks - understanding containerd runc oci etc... ](https://www.tutorialworks.com/difference-docker-containerd-runc-crio-oci/)
