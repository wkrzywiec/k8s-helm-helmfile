# Kubernetes Deployment

### Table of contents 
- [Kubernetes Deployment](#kubernetes-deployment)
    - [Table of contents](#table-of-contents)
  - [Add postgres & adminer](#add-postgres--adminer)
    - [postgres](#postgres)
    - [adminer](#adminer)
  - [Add kanban](#add-kanban)
  - [Add & config Ingress](#add--config-ingress)
    - [Add Ingress](#add-ingress)
      - [Routing config of Ingress Controller](#routing-config-of-ingress-controller)
  - [Testing application](#testing-application)
    - [Adminer App](#adminer-app)
    - [Kanban App](#kanban-app)
  - [References](#references)


## Add postgres & adminer

### postgres

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

### adminer 


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

## Add kanban 


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

## Add & config Ingress
### Add Ingress 

#### Routing config of Ingress Controller

Now we need to set up some routing rules for *Ingress* controller. Therefore the `ingress-controller.yaml` file has been created and applied:

```bash
$ kubectl apply -f ingress-controller.yaml
ingress.networking.k8s.io/ingress-controller created
```

To enter either adminer or kanaban app run following command:

```
minikube tunnel
```

## Testing application
### Adminer App


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

### Kanban App

The UI is available under the address *http://kanban.k8s.com*.

If you want to connect directly to the backend you need to append above URL with */api/* prefix, e.g. *http://kanban.k8s.com/api/swagger-ui.html*.


## References


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

