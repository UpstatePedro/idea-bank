# Kubernetes for ML

## Kubernetes

Kubernetes (k8s) is a ***container orchestration engine***. It manages & simplifies the automatic deployment, monitoring, scaling & management of containerised applications to a computing cluster.

Key aspects of the k8s operating philosophy:

1. K8s object model: k8s defines a raft of abstractions for thinking about & defining how we want our application deployments to behave
2. Declarative management: k8s follows a declarative programming paradigm whereby users define their target state and k8s is responsible for ensuring that the system reflects this as closely as possible at all times

## Some background on containerisation

**Dedicated servers**

The traditional approach to deploying an application was to install it on its own server / virtual machine.

Each server was run for a specific function(s) with:

- Application code
- Dependencies
- Kernel
- Hardware

This was:
- Slow to deploy / modify (~weeks)
- Inefficient utilisation or resource
- Not portable

![Single server deployment](assets/singleserver.svg)

**Virtual machines**

Virtualisation enabled multiple virtual servers to run on the same physical machine.

- Application code
- Dependencies
- Kernel
- Hardware + hypervisor

(hypervisor = software layer that breaks the coupling between the OS and the underlying hardware; enables multiple OS to share the same hardware)

Improvement:
- Faster deployment (~minutes)
- Improved utilisation
- VMs can be imaged & thereby become more portable

Remaining shortcomings:
- Slow boot-time
- Dependencies are not isolated, resources are not isolated. Interactions between applications running on the same physical machine can cause problems.

These dependency / isolation issues can be solved by running separate VMs for each application.
Duplicate OS on each VM, redundancy & slow boot times

![Virtual machine deployment](assets/virtualmachine.svg)

**Containers**

Instead of this, we can containerise:
- virtualise the user space & run a single kernel
- bundle application code & dependencies within each user space to avoid interactions between the code / dependencies in each space

![Containerised deployments](assets/containerisation.svg)

Containers **don't** carry a full OS and are therefore lighter weight.
They also don't need to boot a full VM and initialise OS for each application, so start & stop times are much faster.

Images are the combination of an application and its dependencies

A container is a running instance of an image.

Docker provides open-source tooling to create & run containers, but it does not help to orchestrate applications at scale.

> This is what kubernetes is used for.


## Kubernetes objects

### The K8s object model

Anything that K8s manages is represented by an object. Each object has a *'kind'* - ie. the object's *type* in K8s vernacular.

For each object, there are two key aspects to the object's state:  
- **Object spec**: desired state
- **Object status**: current state

The K8s ***control plane*** is responsible for running a loop to compare the status of each object against its spec, and taking corrective action where they differ.

### K8s object kinds

pod
namespace
service
ingress
deployment
operators
registry

### Pods

- Smallest *deployable* entity in K8s (you can't deploy a standalone container outside of a pod)
- 1 or more container (tightly coupled ie. share network & storage)
- Each pod has a unique IP address in the cluster
- Containers in a pod can communicate over localhost
- Pods are not self-healing; we need to use other objects to achieve this


## The Kubernetes "control-plane"

```{important}
The **Kubernetes control plane** is a collection of processes that collaborate to perform kubernetes' role as orchestrator of the cluster.
```

![The k8s control plane](assets/k8s.svg)

In any given k8s cluster, there will be a master node, and worker nodes.

> Kubernetes is 'given' a cluster to work with, it doesn't independently provision/remove nodes. Managed services (like GKE) can do this in addition to offering k8s orchestration.

> Node pools are also a GKE concept

On the master, will be:
- kube-apiserver: handles all commands to view / modify the cluster
    - All `kubectl` commands are addressed to the api server
- etcd: the cluster's database: stores all cluster state, including configuration data
    - Only the apiserver ever interacts directly with etcd
- kube-scheduler: selects which node to assign pods to by writing the node name to the pod object (but doesn't actually launch them)
- kube-controller-manager: continuously monitors the state of the cluster (through apiserver)
    - many individual k8s objects are maintained by loops of code called "controllers"
    - certain types of controllers are used to manage workloads

A cloud-specific controller manager can also be added to interact with the services & resources that are specific to particular cloud provider.

On all other nodes, you'll have:
- kubelet: effectively the k8s agent on each node
    - starts pods using the container runtime (containerd (runtime component of Docker) in the case of GKE)
    - monitors pods throughout their lifecycle
- kube-proxy: maintains network connectivity between pods in the cluster

> **affinity** & **anti-affinity** specs encourage pods to run on the same / different nodes

> etcd is a highly durable, distributed key-value store

GKE abstracts away the master node; the user just gets an IP address for the cluster & directs all kubectl commands to that IP.