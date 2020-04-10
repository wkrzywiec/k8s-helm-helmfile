# Kubernetes vs Helm vs Helmfile Deployment comparison

With this project I want to compare 3 approaches of deploying same applictions to Kubernetes cluster:

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

And here is a simpliefied schema of what I would like to achieve:

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
😄  minikube v1.8.1 on Ubuntu 18.04
✨  Automatically selected the docker driver
🔥  Creating Kubernetes in docker container with (CPUs=2) (8 available), Memory=2200MB (7826MB available) ...
🐳  Preparing Kubernetes v1.17.3 on Docker 19.03.2 ...
    ▪ kubeadm.pod-network-cidr=10.244.0.0/16
❌  Unable to load cached images: loading cached images: stat /home/wojtek/.minikube/cache/images/k8s.gcr.io/kube-proxy_v1.17.3: no such file or directory
🚀  Launching Kubernetes ... 
🌟  Enabling addons: default-storageclass, storage-provisioner
⌛  Waiting for cluster to come online ...
🏄  Done! kubectl is now configured to use "minikube"
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
🌟  The 'ingress' addon is enabled
```

### Edit hosts file
As I want to have two different URLs to enter the *adminer* (database management tool) and *kanban* app you need to config your **hosts** file - add following lines:

```
<MINIKUBE_IP>	adminer.k8s.com
<MINIKUBE_IP>	kanban.k8s.com
```

A value for `<MINIKUBE_IP>` placeholder is individual per machine. To figure it out you need to have `minikube` up and running. If you're sure it's working you can run the command:
```bash
$ minikube ip
172.17.0.2
```

So in my case, I need to add following lines to the *hosts*  file:
```
172.17.0.2	adminer.k8s.com
172.17.0.2	kanban.k8s.com
```

Location of *hosts* file on different OS:
* [Linux (Ubuntu)](http://manpages.ubuntu.com/manpages/trusty/man5/hosts.5.html)
* [Windows 10](https://www.groovypost.com/howto/edit-hosts-file-windows-10/)
* [Mac](https://www.imore.com/how-edit-your-macs-hosts-file-and-why-you-would-want#page1)

## Maintenance

Official **kubectl** cheatsheet:

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

Minikube provides a **Dashboard** for entire cluster, after typing following command it will open
```bash
$ minikube dashboard
🔌  Enabling dashboard ...
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:45807/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
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