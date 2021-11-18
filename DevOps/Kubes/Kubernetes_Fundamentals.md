# Kubernetes fundamentals
- [Kubernetes fundamentals](#kubernetes-fundamentals)
  - [What is Kubernetes](#what-is-kubernetes)
  - [Kubernetes Architecture](#kubernetes-architecture)
    - [Nodes](#nodes)
    - [Control Plane](#control-plane)
    - [Kubectl](#kubectl)
  - [Managed vs Self-Hosted](#managed-vs-self-hosted)
  - [Typical Plugins](#typical-plugins)
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

There are two types of nodes within a kubernetes cluster. The **master** nodes contain the control plane components which manage the cluster and schedule containers (pods), and the **worker** nodes which host the pods the comprise the application workload.

Typically a production cluster will have several nodes; however in a learning or resource-limited environment, it is possible to have only one node.

Each node within a kubernetes cluster holds the following components which facilitate the kubernetes runtime environment:

- Kubelet - The kubelet is responsible for ensuring containers are running on the node as expected. The kubelet recieves a set of PodSpecs through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. Note - the kubelet doesn't manage containers which were not created directly by Kubernetes.

- Kube-proxy - kube-proxy maintains network rules on nodes. These network rules allow network communication to Pods from network sessions inside or outside of the cluster. kube-proxy uses the operating system packet filtering layer if there is one and it's available, otherwise, kube-proxy forwards the traffic itself.

- Container Runtime - The container runtime is the software responsible for running containers. Kubernetes supports multiple container runtimes such as Docker, Containerd, RKT and CRI-O. Note - Docker support will be dropped in future in favour of runtimes the use the Container Runtime Interface (CRI) such as Containerd. as Docker was created before kubernetes it is not CRI complient, however this wont impact the way end users interact with kubernetes clusters as images built by docker will still work. -see  https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/
### Control Plane
As well as the above components, the master node/s hold the cluster control plane components which manage the worker nodes and the scheduling of pods:

- etcd - etcd is a consistent and highly-available key value store used as Kubernetes' backing store for all cluster data. As etcd contains all the information within the cluster, access to etcd is equivilent to root/cluster admin within the cluster. Only the Kube API server should have access to ETCD.
  
- Kube-Api Server - The core of the control plane. The API server exposes the kubernetes API which end users, worker nodes and various components interact with. The API can be communicated with via REST calls, or the Kubectl command line tool. The kube-api can scale horizontally by deploying more api instances and load balancing traffic between them.
  
- Kube-Controller-Manager - The brain behind orchestration - maintains a set of controllers for managing various aspects of the cluster such as, node controller, job controller and service account/token controllers.
  
- Kube-Scheduler - Distributes the containers accross the nodes by scheduling activities such as containers to the worker nodes based on events occurring on etcd.

### Kubectl

Kubectl (kube control) is the kube command line tool that provides end users control of Kubernetes clusters. For configuration, kubectl looks for a file named config in the `$HOME/.kube` directory. It is also possible to specify other kubeconfig files by setting the KUBECONFIG environment variable or by setting the --kubeconfig flag.

Kubectl supports multiple commands a cheatsheet for useage can be found here - https://kubernetes.io/docs/reference/kubectl/cheatsheet/

## Managed vs Self-Hosted
Kubernetes is open source and can be deployed on traditional on-premises networks, in hybrid environments, or on public and private cloud infrastructure. 

A Managed Kubernetes deployment is one in which a third-party vendor (such as a cloud provider) take over responsibility for some or all of the work necessary for the set-up and operation of K8s. Depending on the vendor, “managed” can refer to anything from dedicated support, to hosting with pre-configured environments, to full hosting and operation.

The most common managed solutions are provided by cloud platforms such as Amazons EKS, Azure AKS and Googles GKE. in these environments the control/management plane is managed by the hosting provider and the administrators only need to manage the worker nodes and application deployments. Additionally each provider offers a fully managed platform such as amazons ECS (Elastic Container Service) in which the infrastructure is completely managed and administrators only deploy the applications.

A self-hosted cluster is one that is managed by the organisation themselves in which the administrator is responsible for both the control plane and the data plane; worker nodes and application deployments. Configuring kubernetes from scratch can be a difficult process, as such there are a number of tools to spin a kubernetes cluster for practicing:

- Minikube
- Kind - Kubernetes in Docker
- Kubeadm
- microk8s
## Typical Plugins

