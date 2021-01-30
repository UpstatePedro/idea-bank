# Kubernetes for ML

image
container
node
cluster
pod
namespace
service
ingress
deployment
operators
registry


### Containers

The traditional approach to deploying an application was to install it on its own server / virtual machine.

**Dedicated servers**

Each server had specific function(s)

- Application code
- Dependencies
- Kernel
- Hardware

Slow to deploy / modify (~weeks)
Inefficient utilisation
Not portable

**Virtual machines**

Virtualisation enabled multiple virtual servers to run on the same physical machine.

- Application code
- Dependencies
- Kernel
- Hardware + hypervisor

(hypervisor = software layer that breaks the coupling between the OS and the underlying hardware; enables multiple OS to share the same hardware)

Faster deployment (~minutes)
Improved utilisation
VMs can be imaged & thereby become more portable.

Boot-time
Dependencies are not isolated, resources are not isolated. Interactions between applications can cause problems.

These dependency / isolation issues can be solved by running separate VMs for each application.

Duplicate OS on each VM, redundancy & slow boot times

Instead of this, we can containerise:
virtualise the user space & run a single kernel
bundles application code & dependencies within each user space to avoid interactions

containers **don't** carry a full OS and are therefore lighter weight
don't need to boot full VM and initialise OS for each application, so start & stop times are much faster

### Container images

Images are the combination of the application and its dependencies

A container is a running instance of an image.

Docker provides open-source tooling to create & run containers, but it does not help to orchestrate applications at scale.
This is what kubernetes is used for.

## Kubernetes

## The Kubernetes "control-plane"

In any given k8s cluster, there will be a master node, and worker nodes
On the master, will be:
- kube-APIserver: handles all commands to view / modify the cluster
    - All `kubectl` commands are addressed to the api server
- etcd: the cluster's database: stores all cluster state, including configuration data
    - Only the api-server ever interacts directly with etcd
- kube-scheduler: selects which node to assign pods to (doesn't actually launch them)
- kube-controller-manager: continuously monitors the state of the cluster (through api-server)
    - many individual k8s objects are maintained by loops of code called "controllers"

On all other nodes, you'll have:
- kubelet

affinity & anti-affinity specs encourage pods to run on the same / different nodes


> etcd is a highly durable, distributed key-value store