# Kubernetes Course 🫰🏾

In this course we will manage some basic concepts about container orchestation tool.

## Minikube configuration 🍷

Minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes. All you need is Docker (or similarly compatible) container or a Virtual Machine environment

### Installation 🩼

To install the latest minikube stable release on x86-64 Linux using binary download. Check the following link on case of diferent machine architecture https://minikube.sigs.k8s.io/docs/start/

```
bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### Machine minimum resources

- 2 CPUs or more
- 2GB of free memory
- 20GB of free disk space
- Internet connection
- Container or virtual machine manager, such as: Docker, QEMU, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMware Fusion/Workstation

### Start your cluster

```
bash
minikube start
```

### 1- Monitoring with prometheus

The first file is our namespace that are a way to organize clusters into virtual sub-clusters. (monitoring/namespaces.yaml)

```
bash
kubectl appliy -f namespace.yaml
namespace/monitoring created ✅
```

Then we must create some role or permissions in that monitoring subcluster that we have already created. This with the objective that prometheus can access to system files such as extract metrics.



