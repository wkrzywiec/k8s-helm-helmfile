# Helm Deployment


Installing Helm - https://helm.sh/docs/intro/install/


### Helm config

Add repository with official stable Helm charts:

```bash
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
"stable" has been added to your repositories
```

## Blueprint Helm Charts

Within this project there are 3 types of Helm charts, which will be used to deploy all Kubernetes objects.

### App Helm Chart

This is a basic Helm chart, necessary to deploy applications (*adminer*, *kanban-ui* & *kanban-app*). In order to run those apps in the Kubernetes cluster we need to create Deployment & Service for each one of them. This chart is taking care of that - it provides a template for doing that.

It was created with following command:
```bash
$ helm create app
Creating app
```

After cleaning the unncessary templates and adding files for Deployment & Service, the chart looks as follows:
```bash
.
├── charts
├── Chart.yaml
├── .helmignore
├── templates
│   ├── deployment.yaml
│   └── service.yaml
└── values.yaml
```

In order to deploy specific application you need to create a YAML file which will override default default values located in the `values.yaml`:

```yaml
app:
  name: app
  group: app
  replicaCount: 1
  container:
    image: add-image-here
    port: 8080
    env: 
      - key: key
        value: value
  service:
    type: ClusterIP
    port: 8080
```

### Postgres Helm chart

This chart is used to create a PostgreSQL database and includes 4 types of Kubernetes objects: Deployment, Service, PersistentVolumeClaim & ConfigMap. 

All templates and defualt `values.yaml` file is located in the `./postgres` folder:

```bash
.
├── charts
├── Chart.yaml
├── .helmignore
├── templates
│   ├── config.yaml
│   ├── deployment.yaml
│   ├── pvc.yaml
│   └── service.yaml
└── values.yaml
```

Similarly to previous example, in order to create all these objects you need to prepare your own YAML file, which follow the structure from `values.yaml`, like for example it's done in `kanban-postgress.yaml` file. 

### Ingress Helm Chart

And the last chart, is responsible for creating and configurating Ingress Controller. 

It creates the default backend service - marked as `dependency` in `Chart.yaml`:

```yaml
dependencies:
  - name: nginx-ingress
    version: 1.36.0
    repository: https://kubernetes-charts.storage.googleapis.com/
```

And the configuration of Ingress Controller is made by combination of `/templates/ingress.yaml` and `values.yaml`. If you want to change the routing you need to override values from relevant file.


### Kanban-app

```bash
$ helm install -f kanban-app.yaml kanban-app ./app
NAME: kanban-app
LAST DEPLOYED: Thu Apr  9 21:38:47 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

$ helm upgrade -f ingress.yaml ingress ./ingress
Release "ingress" has been upgraded. Happy Helming!
NAME: ingress
LAST DEPLOYED: Thu Apr  9 21:41:57 2020
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
```

### kanban-ui

```bash
$ helm install -f kanban-ui.yaml kanban-ui ./app
NAME: kanban-ui
LAST DEPLOYED: Thu Apr  9 21:46:42 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None


$ helm upgrade -f ingress.yaml ingress ./ingress
Release "ingress" has been upgraded. Happy Helming!
NAME: ingress
LAST DEPLOYED: Thu Apr  9 21:46:58 2020
NAMESPACE: default
STATUS: deployed
REVISION: 3
TEST SUITE: None
```

Reference
=========

https://helm.sh/docs/intro/quickstart/
https://github.com/alexellis/helm3-expressjs-tutorial
https://www.civo.com/learn/guide-to-helm-3-with-an-express-js-microservice

https://codefresh.io/docs/docs/new-helm/helm-best-practices/
https://akomljen.com/package-kubernetes-applications-with-helm/

https://itnext.io/drastically-improve-your-kubernetes-deployments-with-helm-5323e7f11ef8

https://www.reddit.com/r/kubernetes/comments/8x3znr/deploying_multiple_similar_applications_with_helm/e20vwfc/
https://www.reddit.com/r/kubernetes/comments/80hmaj/helm_at_reddit_from_local_dev_staging_to/

