# 1. Kubernetes Fundamentals
- [1. Kubernetes Fundamentals](#1-kubernetes-fundamentals)
  - [1.1. What is Kubernetes](#11-what-is-kubernetes)
  - [1.2. Kubernetes Architecture](#12-kubernetes-architecture)
    - [1.2.1. Nodes](#121-nodes)
    - [1.2.2. Control Plane](#122-control-plane)
    - [1.2.3. Pods](#123-pods)
    - [1.2.4. Kubernetes Namespaces](#124-kubernetes-namespaces)
    - [1.2.5. Kubectl](#125-kubectl)
  - [1.3. Managed vs Self-Hosted](#13-managed-vs-self-hosted)
  - [1.4. Understanding Objects](#14-understanding-objects)
    - [1.4.1. Deployments](#141-deployments)
    - [1.4.2. ReplicaSet](#142-replicaset)
    - [1.4.3. DaemonSets](#143-daemonsets)
    - [1.4.4. StatefulSets](#144-statefulsets)
    - [1.4.5. Cronjobs](#145-cronjobs)
  - [1.5. Networking 101](#15-networking-101)
    - [1.5.1. Services](#151-services)
      - [1.5.1.1. NodePort](#1511-nodeport)
      - [1.5.1.2. ClusterIP](#1512-clusterip)
      - [1.5.1.3. LoadBalancer](#1513-loadbalancer)
  - [1.6. Ingress & Ingress Controller](#16-ingress--ingress-controller)
  - [1.7. Security Concepts](#17-security-concepts)
    - [1.7.1. Pod Security Policies](#171-pod-security-policies)
    - [1.7.2. Admission Controllers](#172-admission-controllers)
    - [1.7.3. Network Policies](#173-network-policies)
    - [1.7.4. Service Mesh](#174-service-mesh)
  - [1.8. References and Links](#18-references-and-links)
## 1.1. What is Kubernetes
TL;DR - Kubernetes, referred to as Kubes or 'k8s', is an open-source container orchestration tool for managing container workloads at scale.

Kubernetes, created by Google, is an open-source platform for managing containerized workloads and services through declarative configuration and automation. 

Containers provide a lightweight platform to bundle applications and microservices. In a production environment, a container orchestration tool such as k8s provides a framework to run distributed systems resiliently, by handling the managent of container Deployments and workloads. For example, if a container dies, k8s will automatically schedule another container to replace it, minimising downtime and reducing the operational overhead on system administrators. Furthermore k8s is able to scale containers up and down based on the load to meet availability needs

Other Orchestration tools:
- Docker Swarm - Easy to setup and start but lacks some of the complex features of k8s
- Apache Mesos - Similar to k8s, difficult to setup and start but provides many features however requires use of frameworks (such as Marathon) to get the benefits. Supports containers and non-containerized applications.
- Hashicorp Nomad - Supports declarative features like Infrastructure as Code (IaC) - Terraform - for deploying applications like Docker containerss
- RedHat OpenShift - Orchestrated and managed by Kubernetes underneath, but provides some additional features such as integrated image registry, source-to-image build and native networking etc.

Kubernetes is the most popular and common container orchestration framework in use.

## 1.2. Kubernetes Architecture

![kubernetes architecture!](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

### 1.2.1. Nodes
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

### 1.2.2. Control Plane
As well as the above components, the master node/s hold the cluster control plane components which manage the worker nodes and the scheduling of pods:

- etcd - etcd is a distributed, consistent and highly-available key value store used as Kubernetes' backing store for all cluster data. As etcd contains all the information within the cluster, access to etcd is equivilent to root/cluster admin within the cluster. Only the Kube API server should have access to ETCD. 
  
- Kube-Api Server - The core of the control plane. The API server exposes the kubernetes API which end users, worker nodes and various components interact with. The API can be communicated with via REST API calls, or by using the Kubectl command line tool. The kube-api can scale horizontally by deploying more API instances and load balancing the traffic between them.
  
- Kube-Controller-Manager - The brain behind orchestration - maintains a set of controllers for managing various aspects of the cluster such as, node controller, job controller and service account/token controllers.
  
- Kube-Scheduler - Distributes the containers accross the nodes by scheduling activities such as containers to the worker nodes based on events occurring on etcd.

### 1.2.3. Pods
Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context. The shared context of a Pod is a set of Linux namespaces, cgroups, and potentially other facets of isolation - the same things that isolate a Docker container.

### 1.2.4. Kubernetes Namespaces
In Kubernetes, namespaces provides a way to isolate groups of resources within a cluster. Namespaces are only applicable to certain 'namespaced' objects (e.g. Deployments, Services, etc) and not for objects which exist cluster-wide (e.g. StorageClass, Nodes, PersistentVolumes, etc).

Namespaces are a way to divide cluster resources between multiple tenants and users and can be used to provide security segregation when different services have different access levels i.e. a user can have access to objects within one namespace but not another.

Kubernetes starts with four initial namespaces:

- Default The default namespace for objects with no other namespace
- Kube-system The namespace for objects created by the Kubernetes system access to this namespace should be restricted.
- Kube-public This namespace is created automatically and is readable by all users (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.
- Kube-node-lease This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure.

### 1.2.5. Kubectl
Kubectl (kube control) is the kube command line tool that provides end users control of Kubernetes clusters. For configuration, kubectl looks for a file named config in the `$HOME/.kube` directory. It is also possible to specify other kubeconfig files by setting the KUBECONFIG environment variable or by setting the --kubeconfig flag.

Kubectl supports multiple commands a cheatsheet for useage can be found here - https://kubernetes.io/docs/reference/kubectl/cheatsheet/

## 1.3. Managed vs Self-Hosted
Kubernetes is open source and can be deployed on traditional on-premise networks, in hybrid environments, or on public and private cloud infrastructure. 

A Managed Kubernetes Deployment is one in which a third-party vendor (such as a cloud provider) manages responsibility for some or all of the work necessary for the set-up and operation of K8s. Depending on the vendor, “managed” can refer to anything from dedicated support, to hosting with pre-configured environments, to full hosting and operation.

The most common managed solutions are those provided by cloud platforms such as Amazon EKS, Azure AKS and Google GKE. In these environments the control/management plane is managed by the hosting provider and the administrators only need to manage the worker nodes and application Deployments. Additionally each provider offers a fully managed platform such as Amazons ECS (Elastic Container Service) in which the infrastructure is completely managed and administrators only deploy the applications.

A self-hosted cluster is one that is managed by the organisation and in which the administrator is responsible for both the control plane and the data plane; master nodes, worker nodes and application Deployments. Configuring kubernetes from scratch can be a difficult process, as such there are a number of solutions that make it quicker and easier to bootstrap a kubernetes cluster, Some common products include:

- Minikube
- Kubernetes in Docker (KIND)
- Kubeadm
- microk8s

Note - for learning kubernetes minikube and Kind allow you to install kubernetes in a Docker container making it easier to spin up, interact with and destroy when finished. To run minikube in docker, install kubectl first (as minikube will automatically populate the kube config) and then install minikube and enable the docker driver `minikube config set driver docker` you can then start a cluster with the command `minikube start` for detailed useage information see - https://minikube.sigs.k8s.io/docs/start/.

## 1.4. Understanding Objects
To deploy microservices and application within k8s you need to define/create an object. You must provide the object spec that describes its desired state as well as some basic information about the object. When you use the Kubernetes API to create the object, that request must include that information as JSON in the request body - typically this is achieved by creating a yaml definition - you provide the information to kubectl via the `.yaml` file and kubectl converts the information to JSON when making the API request.

Useful VSCode extensions for working with K8s objects:
- Kubernetes - syntax highlighting, intellisense etc...
- YAML (RedHat) - YAML validation and auto-completion
- Kubernetes Snippets - code snippets for various k8s Deployments making it faster to deploy resources

### 1.4.1. Deployments
https://kubernetes.io/docs/concepts/workloads/controllers/Deployment/

You shouldn't create pods directly within kubernetes, instead you would use a Deployment (or statefulset etc.) A Deployment provides declarative updates for Pods and ReplicaSets - you describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. 

You can define Deployments to create new Pods/ReplicaSets, remove existing Deployments and roll-back to existing states. 

### 1.4.2. ReplicaSet
https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

A ReplicaSet ensures a specified number of replica Pods are running at any given time by creating and deleting Pods as required to reach the declared number. When a ReplicaSet needs to create new Pods, it uses its Pod template. ReplicaSets are usually defined by a Deployment.

### 1.4.3. DaemonSets
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. This is for usecases such as running a logging and monitoring solution on each node to collect metrics. As nodes are added and removed from the cluster, Pods are added and deleted from them. Deleting a DaemonSet will clean up all the Pods it created.

### 1.4.4. StatefulSets
https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. However, unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling. This is useful for situations where it is required to use persistant storage; if a pod in a StatefulSet fails, the persistent Pod identifiers match existing volumes to the new Pods to replace any that have failed.

### 1.4.5. Cronjobs
https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/

A CronJob creates Jobs on a repeating schedule, where one CronJob object is like one line of a crontab. It runs a job periodically on a given schedule, written in Cron format.

## 1.5. Networking 101
https://kubernetes.io/docs/concepts/cluster-administration/networking/

Networking in kubernetes can be a complex topic. Fundamentally k8s does not provide a native networking solution itself and this should be provided by installing a Container Network Interface (CNI) plugin. There are multiple CNI's available which have different benefits but some common options include:

- Calico - https://docs.projectcalico.org/
- Cilium - https://github.com/cilium/cilium
- Flanel - https://github.com/coreos/flannel#flannel
- Kube-Router - https://github.com/cloudnativelabs/kube-router
- Also each cloud hoster offers its own plugin: 
  - AWS VPC CNI 
  - Azure CNI
  - Google Compute Engine

 Typicall K8s has a flat network, where Pods have their own network and recieves it's own IP address, all pods/containers can communicate with each other without NAT (containers within the same pod can communicate over localhost), and all nodes and services can communicate with pods/containers (and vice-versa) without NAT - The CNI plugins should meet these requirements.

### 1.5.1. Services
https://kubernetes.io/docs/concepts/services-networking/service/

The service abstraction provides a way to expose/loadbalance applications within a cluster both internally, e.g. A frontend application communicating with a backend application, or externally e.g. users accessing an application.

There are a few different service types to facilitate this:
#### 1.5.1.1. NodePort 
Exposes the Service on each Node's IP at a static port (the NodePort). A ClusterIP Service, to which the NodePort Service routes, is automatically created. Users can then access applications via the NodePort Service, from outside the cluster, by requesting the`<NodeIP>:<NodePort>`. There are three ports that must be specified within the service manifest:
- The 'Target port' - the port on the pod where the application is listening.
- The 'Port' - the ClusterIP port that communicates with the target port
- The 'NodePort' - the port that is exposed on the node to externally facilitate access. 

#### 1.5.1.2. ClusterIP 
Exposes the Service on a virtual cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. ClusterIP provides a way for pods to easily communicate within the cluster. I.e in a scenario where there are multiple front and backend pods, a pod can forward the request to the ClusterIP which will forward the request to the appropriate pod.

#### 1.5.1.3. LoadBalancer
Exposes a Service externally by provisioning a cloud provider's load balancer e.g. Elastic Load Balancer. The load balancer will then automatically balance traffic to the available NodePorts. Note this feature only works with supported cloud platforms.

## 1.6. Ingress & Ingress Controller
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource. An Ingress may be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL / TLS, and offer name-based virtual hosting. 

An Ingress controller is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure an edge router or additional frontends to help handle the traffic.

An Ingress does not expose arbitrary ports or protocols. Exposing services other than HTTP and HTTPS to the internet typically uses a Service.

## 1.7. Security Concepts
Kubernetes security can be considered in layers using the 4C's model - Cloud, Cluster, Container, Code - where each layer builds upon the next. Full information can be found at: https://kubernetes.io/docs/concepts/security/overview/

### 1.7.1. Pod Security Policies
https://kubernetes.io/docs/concepts/policy/pod-security-policy/

PSP's were an admission controller that provided the main security controls for pod specification. The PSP could be attached to pods to govern sensitive aspects, such as what mounts, linux capabilities and UID's the pods could be assigned. This feature is now deprecated as of Kubernetes v1.21, and will be removed in v1.25. https://kubernetes.io/blog/2021/04/06/podsecuritypolicy-deprecation-past-present-and-future/

There are several external admission controllers which can be used to provide PSP type controls the most popular options are:
- [OPA/Gatekeeper](https://github.com/open-policy-agent/gatekeeper/)
- [Kyverno](https://github.com/kyverno/kyverno/)
- [K-Rail](https://github.com/cruise-automation/k-rail)

### 1.7.2. Admission Controllers
https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/

An admission controller is a piece of code that intercepts requests to the Kubernetes API server prior to persistence of the object, but **after** the request is authenticated and authorized.

Admission controllers can be either "validating", "mutating", or both.  Mutating controllers can modify the objects they admit; validating controllers may not.

The admission control process proceeds in two phases. In the first phase, mutating admission controllers are run. In the second phase, validating admission controllers are run. If any of the controllers in either phase reject the request, the entire request is rejected immediately and an error is returned to the end-user.

### 1.7.3. Network Policies
NetworkPolicies are an application-centric construct which allow you to specify how a pod is allowed to communicate with various network "entities" over the network. By default, if no policies exist in a namespace, then all ingress and egress traffic is allowed to and from pods in that namespace. 

The entities that a Pod can communicate with are identified through a combination of the following 3 identifiers:
- Other pods that are allowed (exception: a pod cannot block access to itself)
- Namespaces that are allowed
- IP blocks (exception: traffic to and from the node where a Pod is running is always allowed, regardless of the IP address of the Pod or the node)

### 1.7.4. Service Mesh
https://platform9.com/blog/kubernetes-service-mesh-a-comparison-of-istio-linkerd-and-consul/

A service mesh isn't a kubernetes construct, but can be used with kubernetes to increase the ability to manage network traffic between services in a safe and reliable way. There are multiple benefits to using a service mesh but enforcing mTLS is the most important.

Common service mesh technologies includ
- Istio
- LinkerD
- Consul

## 1.8. References and Links
https://techbeacon.com/security/8-open-source-kubernetes-vulnerability-scanners-consider

