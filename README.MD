# Kubernetes Course 🫰🏾

In this course we will manage some basic concepts about container orchestation tool.

## Minikube configuration 🍷

Minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes. All you need is Docker (or similarly compatible) container or a Virtual Machine environment

### Installation 🐙

To install the latest minikube stable release on x86-64 Linux using binary download. Check the following link on case of diferent machine architecture https://minikube.sigs.k8s.io/docs/start/

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### Machine minimum resources 🎁

- 2 CPUs or more
- 2GB of free memory
- 20GB of free disk space
- Internet connection
- Container or virtual machine manager, such as: Docker, QEMU, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMware Fusion/Workstation

### Start your cluster 🚀

```
minikube start
```

### 1- Monitoring and notifications to an Slack channel 🎠

Prometheus server is responsible for send alerts of monitoring to the alert manager and this one is in charge to alert to our different integrations suach send email or slack etc.

The first file is our namespace that are a way to organize clusters into virtual sub-clusters. (monitoring/namespaces.yaml)

```
kubectl appliy -f namespace.yaml
namespace/monitoring created ✅
```

#### Prometheus Pod

Then we must create some role or permissions in that monitoring subcluster that we have already created. This with the objective that prometheus can access to system files such as extract metrics.

```
kubectl apply -f prometheus-cluster-role.yaml
clusterrole.rbac.authorization.k8s.io/prometheus created ✅
clusterrolebinding.rbac.authorization.k8s.io/prometheus created ✅
```

Also, we need to deploy our prometheus pod server that it will create our container insade base on the following image **prom/prometheus:v2.1.0** in ``monitoring``namespace and it will attach some config and storage volume in the root cluster.

```
kubectl apply -f prometheus-cluster-role.yaml
deployment.apps/prometheus-deployment created ✅
```

Check what we have iat this moment:

```
kubectl -n monitoring get all
```

|NAME                                  | READY|   STATUS           |RESTARTS|   AGE|
|-------------------------------------|------|--------------------|--------|------|
|pod/prometheus-deployment-5875b79765-nzpjp|   0/1|     ContainerCreating|   0|          2m31s|
|deployment.apps/prometheus-deployment|   0/1|     1|           0|          2m31s|
|replicaset.apps/prometheus-deployment-5875b79765|   1|         1|         0|       2m31s|


*Config maps* are manifiest that you inject to an specific namespace and then whatever container running on this ns can read those configuration maps and generated its configuration base on that template (Global Enviroment Namespaces)

**Basic Check of Memory Use**
```
rules:
      - alert: High Pod Memory
        expr: sum(container_memory_usage_bytes) > 1
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: High Memory Usage
```

Applying Map Configuration:

```
kubectl apply -f prometheus-configmap.yaml
configmap/prometheus-server-conf created ✅
```

Creation of a service to expose our prometheus pods using a node port that basiclly open a port in the workers nodes that redirects the traffic to the pod with specific label and selector.

```
kubectl apply -f prometheus-service.yaml 
service/prometheus-service created ✅
```

#### Alert Manager Pod




