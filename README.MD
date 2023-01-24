# Kubernetes Course 🫰

In this course we will manage some basic concepts about container orchestation tool.

## Minikube configuration 🍷

Minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes. All you need is Docker (or similarly compatible) container or a Virtual Machine environment

## Installation 🐙

To install the latest minikube stable release on x86-64 Linux using binary download. Check the following link on case of diferent machine architecture https://minikube.sigs.k8s.io/docs/start/

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## Machine minimum resources 🎁

- 2 CPUs or more
- 2GB of free memory
- 20GB of free disk space
- Internet connection
- Container or virtual machine manager, such as: Docker, QEMU, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMware Fusion/Workstation

## Start your cluster 🚀

```
minikube start
```

## 1- Monitoring and notifications to an Slack channel 🎠

Prometheus server is responsible for send alerts of monitoring to the alert manager and this one is in charge to alert to our different integrations suach send email or slack etc.

The first file is our namespace that are a way to organize clusters into virtual sub-clusters. (1-monitoring/namespaces.yaml)

```
kubectl appliy -f namespace.yaml
namespace/monitoring created ✅
```

### Prometheus Pod  🚂

Then we must create some role or permissions in that monitoring subcluster that we have already created. This with the objective that prometheus can access to system files such as **extract metrics**.

```
kubectl apply -f prometheus-cluster-role.yaml
clusterrole.rbac.authorization.k8s.io/prometheus created ✅
clusterrolebinding.rbac.authorization.k8s.io/prometheus created ✅
```

Also, we need to deploy our prometheus pod server that it will create our container instance base on the following image **prom/prometheus:v2.1.0** in ``monitoring``namespace and it will attach some config and storage volume in the root cluster.

```
kubectl apply -f prometheus-cluster-role.yaml
deployment.apps/prometheus-deployment created ✅
```

Check what we have at this moment:

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
```yaml
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

Creation of a service to expose our prometheus pods using a node port that basically open a port in the workers nodes that redirects the traffic to the pod with specific label or selector.

```
kubectl apply -f prometheus-service.yaml 
service/prometheus-service created ✅
```

### Alert Manager Pod 🪝

- Service creation (Cluster Ip because we want that prometheus can access to that container assigning an specific ip in the cluster)

    ```
    kubectl apply -f alert-manager-service.yaml
    service/alertmanager created ✅
    ```

- Deployment in this case we have a pod that will have two containers, the firts one is the alert manager and the second one a **side car** that will be watching for changes in global config map. Also, we have a Persistent Volume Claim that is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources, claims can request specific size and access modes.

    ```
    kubectl apply -f alert-manager-configmap.yaml 
    service/alertmanager created ✅

    kubectl apply -f alert-manager-pvc.yaml 
    persistentvolumeclaim/alertmanager created ✅

    kubectl apply -f alert-manager-deployment.yaml 
    deployment.apps/alertmanager created ✅
    ```

### Checking all infrastructure 🏗️

- Pods

|NAME                                  | READY|   STATUS           |RESTARTS|   AGE|
|-------------------------------------|------|--------------------|--------|------|
|pod/alertmanager-85bc4f4759-s25c5|            2/2|     Running|   0|         23m|
|pod/prometheus-deployment-5875b79765-nzpjp|   1/1|     Running|   0|         91m|

- Services

|NAME                         |TYPE        |CLUSTER-IP |    EXTERNAL-IP|   PORT(S)|          AGE|
|-------------------------------------|------|--------------------|--------|------|--------------|
|service/alertmanager|         ClusterIP|   10.110.82.83|   <none>|        9093/TCP|         59m|
|service/prometheus-service|   NodePort|    10.100.0.75|    <none> |       8080:30000/TCP|   79m|

- Deployments

|NAME                                  |  READY  | UP-TO-DATE |  AVAILABLE |  AGE|
|-------------------------------------|------|--------------------|--------|------|
|deployment.apps/alertmanager|            1/1 |    1    |        1   |        23m|
|deployment.apps/prometheus-deployment|   1/1 |    1  |          1  |         91m|

### Considerations 🚦

Due to we are using minikube for our local kubernetes cluster we just have one node and for accesing externally we must use the following command:

```bash
minikube -n monitoring service prometheus-service --url
http://192.168.49.2:30000 ⚠️
```

However i have my minikube configuration inside an ec2 vm hoste in aws cloud provider 😅. That is why i must port forward my vm local entry set in 31000 port and the service port 8080 of th prometheus node port:

```bash
kubectl port-forward --address 0.0.0.0 service/prometheus-service --namespace monitoring 31000:8080
Forwarding from 0.0.0.0:31000 -> 9090
Handling connection for 31000
```

### Evidences 🚧

![Graphs](./assets/graph-prometheus.png)

![Alerts](./assets/alerts-prometheus.png)

![Slack-Configuration](./assets/slack-config.png)

![Slack-Evidence](./assets/slack-evidence.png)