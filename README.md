# Kubernetes vs Helm vs Helmfile Deployment comparison

With this project I want to compare 3 approaches of deploying same applications to Kubernetes cluster:

* **k8s** - the entire deployment is done with `kubectl` - Kubernetes command line tool,
* **helm** - the deployment is done by using [Helm charts](https://helm.sh),
* **helmfile** - very similar to previous one, but this time with installed helmfile plugin for Helm.

## Usage

Enter one of the folders to find out about one of the approaches.

## Project architecture

This project is based on my previous one - [Kanban Board](https://github.com/wkrzywiec/kanban-board) *(source code)*. 

It contains 3 components:
* postgres - database
* kanban-app - backend service, serving REST endpoints for a frontend
* kanban-ui - frontend service

And here is a simplified schema of what I would like to achieve:

![Simple Architecture Diagram](https://github.com/wkrzywiec/k8s-helm-helmfile/blob/master/assets/arch-simple.png)


On it you there is an additional component - adminer. It's GUI application for managing the database.

A full picture of Kubernetes cluster that is created with each approach is presented below:

![Kubernetes Objects Architecture](https://github.com/wkrzywiec/k8s-helm-helmfile/blob/master/assets/arch-k8s.png)

## Prerequisites

Before testing any of described approaches you need first go through following steps:

### Install Minikube & Kubectl
* Installing Docker - https://docs.docker.com/install/
* Installing minikube - https://kubernetes.io/docs/tasks/tools/install-minikube/
* Installing kubectl - https://kubernetes.io/docs/tasks/tools/install-kubectl/

### Start a new Minikube cluster

In order to run a minikube cluster:
```bash
$ minikube start
😄  minikube v1.25.2 on Ubuntu 20.04 (amd64)
✨  Automatically selected the docker driver
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

To check the status of the cluster:
```bash
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

To check that `kubectl` is properly configured:
```bash
$ kubectl cluster-info
Kubernetes master is running at https://127.0.0.1:32768
KubeDNS is running at https://127.0.0.1:32768/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Next, we need to run another command to enable *Ingress* addon:
```bash
$ minikube addons enable ingress
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.1.1
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
``` 

### Edit hosts file
As I want to have two different URLs to enter the *adminer* (database management tool) and *kanban* app you need to config your **hosts** file - add following lines:

```
127.0.0.1	adminer.k8s.com
127.0.0.1	kanban.k8s.com
```

Location of *hosts* file on different OS:
* [Linux (Ubuntu)](http://manpages.ubuntu.com/manpages/trusty/man5/hosts.5.html)
* [Windows 10](https://www.groovypost.com/howto/edit-hosts-file-windows-10/)
* [Mac](https://www.imore.com/how-edit-your-macs-hosts-file-and-why-you-would-want#page1)

To access one of these addresses one last thing is needed - running following command:

```
$ minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

```

## Maintenance

Official **kubectl** cheatsheet:

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

Minikube provides a **Dashboard** for entire cluster, after typing following command it will open
```bash
$ minikube dashboard
🔌  Enabling dashboard ...
    ▪ Using image kubernetesui/dashboard:v2.3.1
    ▪ Using image kubernetesui/metrics-scraper:v1.0.7
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:46801/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
👉  http://127.0.0.1:46801/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

To see a resource (CPU, memory) consumption of services you can enable *metrics-server* minikube addon (they will be visible on a dashboard):
```bash
$ minikube addons enable metrics-server
    ▪ Using image k8s.gcr.io/metrics-server/metrics-server:v0.4.2
🌟  The 'metrics-server' addon is enabled
```

If your pod is not starting properly you can investigate it by **describe pods <pod-name>** command:
```bash
$ kubectl describe pods postgres-6fd67d4976-ljd2j
Name:         postgres-6fd67d4976-ljd2j
Namespace:    default
Priority:     0
Node:         m01/172.17.0.2
Start Time:   Tue, 24 Mar 2020 07:50:28 +0100
Labels:       app=postgres
              pod-template-hash=6fd67d4976
              type=db
Annotations:  <none>
Status:       Pending
IP:           172.18.0.4
IPs:
  IP:           172.18.0.4
Controlled By:  ReplicaSet/postgres-6fd67d4976
Containers:
  postgres:
    Container ID:   
    Image:          postgres:9.6-alpine
    Image ID:       
    Port:           5432/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       CreateContainerConfigError
    Ready:          False
    Restart Count:  0
    Environment:
      POSTGRES_DB:        kanban
      POSTGRES_USER:      kanban
      POSTGRES_PASSWORD:  kanban
    Mounts:
      /var/lib/postgresql/data from postgres-storage (rw,path="postgres")
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-nlb25 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  postgres-storage:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  postgres-persistent-volume-claim
    ReadOnly:   false
  default-token-nlb25:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-nlb25
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  2m3s                default-scheduler  Successfully assigned default/postgres-6fd67d4976-ljd2j to m01
  Normal   Pulled     6s (x11 over 2m2s)  kubelet, m01       Container image "postgres:9.6-alpine" already present on machine
  Warning  Failed     6s (x11 over 2m2s)  kubelet, m01       Error: stat /tmp/hostpath-provisioner/pvc-f5d9b781-9cdf-4a4c-8c9b-2edb8330d139: no such file or directory

```