# Kubernetes Deployment

### Table of contents 
* [Install Minikube & Kubectl](#Install-Minikube-&-Kubectl)
* [Start a new Minikube cluster](#Start-a-new-Minikube-cluster)
* [Edit hosts file](#Edit-hosts-file)
* [Add postgres & adminer](#Add-postgres-&-adminer)
  * [postgres](#postgres)
  * [adminer](#adminer)
* [Add kanban](#Add-kanban)
* [Add & config Ingress](#Add-&-config-Ingress)
  * [Add Ingress](#Add-Ingress) 
  * [Routing config of Ingress Controller](#Routing-config-of-Ingress-Controller)
* [Testing application](#Testing-application)
  * [Adminer App](#Adminer-App)
  * [Kanban App](#Kanban-App)
* [Maintenance](#Maintenance)
* [References](#References)

Install Minikube & Kubectl
==========================

Installing Docker - https://docs.docker.com/install/
Installing minikube - https://kubernetes.io/docs/tasks/tools/install-minikube/
Installing kubectl - https://kubernetes.io/docs/tasks/tools/install-kubectl/

Start a new Minikube cluster
============================

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

Edit hosts file
===============

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

Add postgres & adminer
======================

postgres
--------


First create [**PeristentVolumeClaim**](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) using `postgres-pvc.yaml` file. And then apply it:
```bash
$ kubectl apply -f postgres-pvc.yaml
persistentvolumeclaim/postgres-persistent-volume-claim created

$ kubectl get pvc
NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-persistent-volume-claim   Bound    pvc-f5d9b781-9cdf-4a4c-8c9b-2edb8330d139   4Gi        RWO            standard       10s
```

Next create a [**ConfigMap**](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) that will hold all database credentials - `postgres-configmap.yaml`:
```bash
$ kubectl apply -f postgres-config.yaml
configmap/postgres-config created

$ kubectl get configmaps
NAME              DATA   AGE
postgres-config   3      11s
```

Then create a new file for [**Deployment**](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) - `postgres-deployment.yaml` and then apply it:
```bash
$ kubectl apply -f postgres-deployment.yaml
deployment.apps/postgres created

$ kubectl get deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
postgres   1/1     1            1           5m6s
```

And finally we need to expose pods from this *Deployment* to other pods inside this cluster using [**ClusterIP**](https://kubernetes.io/docs/concepts/services-networking/service/) Service - `postgres-svc.yaml`:
```bash
$ kubectl apply -f postgres-svc.yaml
service/postgres created

$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    8m35s
postgres     ClusterIP   10.109.98.250   <none>        5432/TCP   4m34s
```

adminer 
-------

Postgres is set up (but it can't be accessable by any other service inside or outside cluster yet). So let's move on to **adminer**.

Similar story here, first we need to create a *Deployment* (`adminer-deployment.yaml`) & *Service* (`adminer-svc.yaml`) for adminer and both of them apply using `kubectl apply -f` command.

```bash
$ kubectl apply -f adminer-deployment.yaml
deployment.apps/adminer created

$ kubectl apply -f adminer-svc.yaml
service/adminer created
```

To check if all your Pods and Deplyments are ok you can type:
```bash
$ kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
adminer-6f9fd8d895-jqjpb   1/1     Running   0          7m14s
postgres-8ff946f87-z4bzn   1/1     Running   0          9m15s

$ kubectl get deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
adminer    1/1     1            1           7m41s
postgres   1/1     1            1           9m42s
```

Add kanban 
==========

First `kanban-app-deployment.yaml` file has beed created. Then inside a terminal use following command:
```bash
$ kubectl apply -f kanban-app-deployment.yaml
deployment.apps/kanban-app-deployment created
```

Now to check if Pod & Deployment has been created:
```bash
$ kubectl get pods
NAME                                     READY   STATUS             RESTARTS   AGE
kanban-app-deployment-67445565b8-vplbq   0/1     CrashLoopBackOff   3          2m10s

$ kubectl get deployments
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
kanban-app-deployment   0/1     1            0           2m56s
```

Add & config Ingress
====================
Add Ingress 
-----------

To get inside the application you need to set up [**Ingress**](https://kubernetes.io/docs/concepts/services-networking/ingress/) which is a gateway to a cluster - the only way you can enter any application inside of it.

Community implementation of *Ingress*: https://github.com/kubernetes/ingress-nginx

Article about *Ingress*: https://www.joyfulbikeshedding.com/blog/2018-03-26-studying-the-kubernetes-ingress-system.html

Accordingly to an [official website documentation](https://kubernetes.github.io/ingress-nginx/deploy/) first we need to run following command in order to install *Ingress*:

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml
namespace/ingress-nginx created
configmap/nginx-configuration created
configmap/tcp-services created
configmap/udp-services created
serviceaccount/nginx-ingress-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-role created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-role-nisa-binding created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
deployment.apps/nginx-ingress-controller created
limitrange/ingress-nginx created
```

For simplicity, I've downloaded above file and save it as *ingress-nginx.yaml*.

Next, we need to run another command to enable *Ingress* addon:
```bash
$ minikube addons enable ingress
🌟  The 'ingress' addon is enabled
```

Routing config of Ingress Controller
------------------------------------

Now we need to set up some routing rules for *Ingress* controller. Therefore the `ingress-controller.yaml` file has been created and applied:
```bash
$ kubectl apply -f ingress-controller.yaml
ingress.networking.k8s.io/ingress-controller created
```

Testing application
===================
Adminer App
-----------

It's available under the URL *http://adminer.k8s.com* and the credentials are:
```
System: PostgreSQL
Server: postgres
Username: kanban
Password: kanban
Database: kanban
```

After that you should be login to the database. 

If you want to change the default design of an adminer you can specify it in the `adminer-deployment.yaml` file in the `spec.template.spec.containers[0].env[0]` where you can change a `value` of `ADMINER_DESIGN` variable from `pepa-linha` to whatever you want (full list available in [the GitHub project](https://github.com/vrana/adminer/tree/master/designs)):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: adminer
spec:
  template:
    spec:
      containers:
          env:
            - name: ADMINER_DESIGN
              value: pepa-linha
```

Kanban App
----------

The UI is available under the address *http://kanban.k8s.com*.

If you want to connect directly to the backend you need to append above URL with */api/* prefix, e.g. *http://kanban.k8s.com/api/swagger-ui.html*.

Maintenance
===========

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

References
==========

* kubectl overview
https://kubernetes.io/docs/reference/kubectl/overview/

* `kubectl create` vs `kubectl apply`
https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create 

* Minikube
https://kubernetes.io/docs/setup/learning-environment/minikube/

* Ingress Controllers
https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

* PostgreSQL in Kubernetes
https://severalnines.com/database-blog/using-kubernetes-deploy-postgresql

* Adminer Docker
https://hub.docker.com/_/adminer/

