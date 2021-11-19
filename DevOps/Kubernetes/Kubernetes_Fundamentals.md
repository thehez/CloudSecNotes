# Kubernetes fundamentals
- [Kubernetes fundamentals](#kubernetes-fundamentals)
  - [What is Kubernetes](#what-is-kubernetes)
  - [Kubernetes Architecture](#kubernetes-architecture)
    - [Nodes](#nodes)
    - [Control Plane](#control-plane)
    - [Pods](#pods)
    - [Kubernetes Namespaces](#kubernetes-namespaces)
    - [Kubectl](#kubectl)
  - [Managed vs Self-Hosted](#managed-vs-self-hosted)
  - [Defining Objects](#defining-objects)
    - [Deployments](#deployments)
    - [ReplicaSet](#replicaset)
    - [DaemonSets](#daemonsets)
## What is Kubernetes
TL;DR - Kubernetes, referred to as Kubes or 'k8s', is an open-source container orchestration tool for managing container workloads at scale.

Kubernetes, created by Google, is an open-source platform for managing containerized workloads and services through declarative configuration and automation. 

Containers provide a lightweight platform to bundle applications and microservices. In a production environment, a container orchestration tool such as k8s provides a framework to run distributed systems resiliently, by handling the managent of container deployments and workloads. For example, if a container dies, k8s will automatically schedule another container to replace it, minimising downtime and reducing the operational overhead on system administrators. Furthermore k8s is able to scale containers up and down based on the load to meet availability needs

Other Orchestration tools:
- Docker Swarm - Easy to setup and start but lacks some of the complex features of k8s
- Apache Mesos - Similar to k8s, difficult to setup and start but provides many features however requires use of frameworks (such as Marathon) to get the benefits. Supports containers and non-containerized applications.
- Hashicorp Nomad - Supports declarative features like Infrastructure as Code (IaC) - Terraform - for deploying applications like Docker containerss
- RedHat OpenShift - Orchestrated and managed by Kubernetes underneath, but provides some additional features such as integrated image registry, source-to-image build and native networking etc.

Kubernetes is the most popular and common container orchestration framework in use.

## Kubernetes Architecture

![kubernetes architecture!](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

### Nodes
A k8s cluster consists of several machines, either physical or virtual, known as nodes. 

There are two types of nodes within a kubernetes cluster. The **master** nodes contain the control plane components which manage the cluster and schedule containers (pods), and the **worker** nodes which host the pods that contains the microservices/application workload.

Typically a production cluster will have several nodes; however in a learning or resource-limited environment, it is possible to have only one node.

Each node within a cluster has the following components which facilitate the kubernetes runtime environment:

- Kubelet - The kubelet is responsible for ensuring containers are running on the node as expected. The kubelet recieves a set of PodSpecs through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. Note - the kubelet doesn't manage containers which were not created directly by Kubernetes.

- Kube-proxy - kube-proxy maintains network rules on nodes. These network rules allow network communication to Pods from network sessions inside or outside of the cluster. kube-proxy uses the operating system packet filtering layer if there is one and it's available, otherwise, kube-proxy forwards the traffic itself.

- Container Runtime - The container runtime is the software responsible for running containers. Kubernetes supports multiple container runtimes such as Docker, Containerd, RKT and CRI-O. Note - Docker support will be dropped in future in favour of runtimes the use the Container Runtime Interface (CRI) such as Containerd. as Docker was created before kubernetes it is not CRI complient, however this wont impact the way end users interact with kubernetes clusters as images built by docker will still work. -see  https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/

There are two ways to add a node to a cluster:

- Manually - An administrator can add a node by deploying a node object through a manifest
- Automatically - Setting the `--register-node` flag on the kubelet will cause the kubelet to attempt to register itself with the kube-api server.
### Control Plane
As well as the above components, the master node/s hold the cluster control plane components which manage the worker nodes and the scheduling of pods:

- etcd - etcd is a consistent and highly-available key value store used as Kubernetes' backing store for all cluster data. As etcd contains all the information within the cluster, access to etcd is equivilent to root/cluster admin within the cluster. Only the Kube API server should have access to ETCD.
  
- Kube-Api Server - The core of the control plane. The API server exposes the kubernetes API which end users, worker nodes and various components interact with. The API can be communicated with via REST calls, or the Kubectl command line tool. The kube-api can scale horizontally by deploying more api instances and load balancing traffic between them.
  
- Kube-Controller-Manager - The brain behind orchestration - maintains a set of controllers for managing various aspects of the cluster such as, node controller, job controller and service account/token controllers.
  
- Kube-Scheduler - Distributes the containers accross the nodes by scheduling activities such as containers to the worker nodes based on events occurring on etcd.

### Pods
Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. The shared context of a Pod is a set of Linux namespaces, cgroups, and potentially other facets of isolation - the same things that isolate a Docker container.

In terms of Docker concepts, a Pod is similar to a group of Docker containers with shared namespaces and shared filesystem volumes.
Using Pod

### Kubernetes Namespaces
In Kubernetes, namespaces provides a way to isolate groups of resources within a cluster. Namespaces are only applicable to certain 'namespaced' objects (e.g. Deployments, Services, etc) and not for objects which exist cluster-wide (e.g. StorageClass, Nodes, PersistentVolumes, etc).
Namespaces are intended for use in environments with many users spread across multiple teams, or projects. For clusters with a few to tens of users, you should not need to create or think about namespaces at all. Start using namespaces when you need the features they provide.

Namespaces are a way to divide cluster resources between multiple tenants and users and can be used to provide security segregation when different services have different access levels i.e. a user can have access to objects within one namespace but not another.

Kubernetes starts with four initial namespaces:

- Default The default namespace for objects with no other namespace
- Kube-system The namespace for objects created by the Kubernetes system access to this namespace should be restricted.
- Kube-public This namespace is created automatically and is readable by all users (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.
- Kube-node-lease This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure.
### Kubectl
Kubectl (kube control) is the kube command line tool that provides end users control of Kubernetes clusters. For configuration, kubectl looks for a file named config in the `$HOME/.kube` directory. It is also possible to specify other kubeconfig files by setting the KUBECONFIG environment variable or by setting the --kubeconfig flag.

Kubectl supports multiple commands a cheatsheet for useage can be found here - https://kubernetes.io/docs/reference/kubectl/cheatsheet/

## Managed vs Self-Hosted
Kubernetes is open source and can be deployed on traditional on-premise networks, in hybrid environments, or on public and private cloud infrastructure. 

A Managed Kubernetes deployment is one in which a third-party vendor (such as a cloud provider) manages responsibility for some or all of the work necessary for the set-up and operation of K8s. Depending on the vendor, “managed” can refer to anything from dedicated support, to hosting with pre-configured environments, to full hosting and operation.

The most common managed solutions are those provided by cloud platforms such as Amazon EKS, Azure AKS and Google GKE. In these environments the control/management plane is managed by the hosting provider and the administrators only need to manage the worker nodes and application deployments. Additionally each provider offers a fully managed platform such as Amazons ECS (Elastic Container Service) in which the infrastructure is completely managed and administrators only deploy the applications.

A self-hosted cluster is one that is managed by the organisation and in which the administrator is responsible for both the control plane and the data plane; master nodes, worker nodes and application deployments. Configuring kubernetes from scratch can be a difficult process, as such there are a number of solutions that make it quicker and easier to bootstrap a kubernetes cluster, Some common products include:

- Minikube
- Kubernetes in Docker (KIND)
- Kubeadm
- microk8s

Note - for learning kubernetes minikube and Kind allow you to install kubernetes in a Docker container making it easier to spin up, interact with and destroy when finished. To run minikube in docker, install kubectl first (as minikube will automatically populate the kube config) and then install minikube and enable the docker driver `minikube config set driver docker` you can then start a cluster with the command `minikube start` for detailed useage information see - https://minikube.sigs.k8s.io/docs/start/.
## Defining Objects
To create an object in Kubernetes, you must provide the object spec that describes its desired state, as well as some basic information about the object. When you use the Kubernetes API to create the object that API request must include that information as JSON in the request body. Typically this is achieved by creating a yaml definition, you provide the information to kubectl via the `.yaml` file and kubectl converts the information to JSON when making the API request.

Useful VSCode extensions for Kubernetes:
- Kubernetes - syntax highlighting, intellisense etc...
- YAML (RedHat) - YAML validation and auto-completion
- Kubernetes Snippets - code snippets for various k8s deployments making it faster to deploy resources

### Deployments
### ReplicaSet
### DaemonSets