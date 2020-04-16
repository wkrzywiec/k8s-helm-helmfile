# Helm Deployment

### Table of content
* [Install Helm](#Install-Helm)
  * [Helm config](#Helm-config)
* [Blueprint Helm Charts](#Blueprint-Helm-Charts)
  * [App Helm Chart](#App-Helm-Chart)
  * [Postgres Helm chart](#Postgres-Helm-chart)
  * [Ingress Helm Chart](#Ingress-Helm-Chart)
* [Create helm releases](#Create-helm-releases)
  * [Postgres](#Postgres)
  * [Adminer](#Adminer)
  * [Kanban-app](#Kanban-app)
  * [Kanban-ui](#Kanban-ui)
  * [Ingress](#Ingress)
* [Maintanance](#Maintanance)
* [References](#References)



## Install Helm
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

Similarly to previous example, to create all these objects you need to prepare your own YAML file, which follow the structure from `values.yaml`, like for example it's done in `kanban-postgress.yaml` file. 

### Ingress Helm Chart

And the last chart, is responsible for creating and configurating Ingress Controller. 

It creates the default backend service - marked as `dependency` in `Chart.yaml`:

```yaml
dependencies:
  - name: nginx-ingress
    version: 1.36.0
    repository: https://kubernetes-charts.storage.googleapis.com/
```

And the configuration of Ingress Controller, which is made by combining of `/templates/ingress.yaml` and `values.yaml`. If you want to change the routing you need to override values from relevant file.

In order to download that dependency use the command:

```
$ helm dependency update ./ingress/
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading nginx-ingress from repo https://kubernetes-charts.storage.googleapis.com/
Deleting outdated charts
```

## Create helm releases

Here is the list of commands that needs to be executed to deploy applications on the cluster. 

### Postgres

```bash
$ helm install -f kanban-postgres.yaml postgres ./postgres
NAME: postgres
LAST DEPLOYED: Fri Apr 10 22:42:44 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### Adminer

```bash
$ helm install -f adminer.yaml adminer ./app
NAME: adminer
LAST DEPLOYED: Fri Apr 10 22:43:19 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### Kanban-app

```bash
$ helm install -f kanban-app.yaml kanban-app ./app
NAME: kanban-app
LAST DEPLOYED: Fri Apr 10 22:43:45 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### Kanban-ui

```bash
$ helm install -f kanban-ui.yaml kanban-ui ./app
NAME: kanban-ui
LAST DEPLOYED: Fri Apr 10 22:44:16 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

### Ingress

```bash
$ helm install -f ingress.yaml ingress ./ingress
NAME: ingress
LAST DEPLOYED: Fri Apr 10 22:44:42 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

## Maintanance

In order to check the list of Helm releases:

```bash
$ helm list
NAME      	NAMESPACE	REVISION	UPDATED             STATUS  	CHART         	APP VERSION
adminer   	default  	1       	2020-04-10 22:43:19	deployed	app-0.1.0     	1.16.0     
ingress   	default  	1       	2020-04-10 22:44:42	deployed	ingress-0.1.0 	1.16.0     
kanban-app	default  	1       	2020-04-10 22:43:45	deployed	app-0.1.0     	1.16.0     
kanban-ui 	default  	1       	2020-04-10 22:44:16 deployed	app-0.1.0     	1.16.0     
postgres  	default  	1       	2020-04-10 22:42:44	deployed	postgres-0.1.0	1.16.0 
```

To update the release just replace the `install` key word in a command with `upgrade`, e.g.: 

```
$ helm upgrade -f ingress.yaml ingress ./ingress
```

To delete the release use:

```bash
$ helm uninstall ingress
```

All other commands are described in official docs or under command:

```bash
$ helm help
The Kubernetes package manager

Common actions for Helm:

- helm search:    search for charts
- helm pull:      download a chart to your local directory to view
- helm install:   upload the chart to Kubernetes
- helm list:      list releases of charts

Environment variables:

+------------------+-----------------------------------------------------------------------------+
| Name             | Description                                                                 |
+------------------+-----------------------------------------------------------------------------+
| $XDG_CACHE_HOME  | set an alternative location for storing cached files.                       |
| $XDG_CONFIG_HOME | set an alternative location for storing Helm configuration.                 |
| $XDG_DATA_HOME   | set an alternative location for storing Helm data.                          |
| $HELM_DRIVER     | set the backend storage driver. Values are: configmap, secret, memory       |
| $HELM_NO_PLUGINS | disable plugins. Set HELM_NO_PLUGINS=1 to disable plugins.                  |
| $KUBECONFIG      | set an alternative Kubernetes configuration file (default "~/.kube/config") |
+------------------+-----------------------------------------------------------------------------+

Helm stores configuration based on the XDG base directory specification, so

- cached files are stored in $XDG_CACHE_HOME/helm
- configuration is stored in $XDG_CONFIG_HOME/helm
- data is stored in $XDG_DATA_HOME/helm

By default, the default directories depend on the Operating System. The defaults are listed below:

+------------------+---------------------------+--------------------------------+-------------------------+
| Operating System | Cache Path                | Configuration Path             | Data Path               |
+------------------+---------------------------+--------------------------------+-------------------------+
| Linux            | $HOME/.cache/helm         | $HOME/.config/helm             | $HOME/.local/share/helm |
| macOS            | $HOME/Library/Caches/helm | $HOME/Library/Preferences/helm | $HOME/Library/helm      |
| Windows          | %TEMP%\helm               | %APPDATA%\helm                 | %APPDATA%\helm          |
+------------------+---------------------------+--------------------------------+-------------------------+

Usage:
  helm [command]

Available Commands:
  completion  Generate autocompletions script for the specified shell (bash or zsh)
  create      create a new chart with the given name
  dependency  manage a chart\'s dependencies
  env         Helm client environment information
  get         download extended information of a named release
  help        Help about any command
  history     fetch release history
  install     install a chart
  lint        examines a chart for possible issues
  list        list releases
  package     package a chart directory into a chart archive
  plugin      install, list, or uninstall Helm plugins
  pull        download a chart from a repository and (optionally) unpack it in local directory
  repo        add, list, remove, update, and index chart repositories
  rollback    roll back a release to a previous revision
  search      search for a keyword in charts
  show        show information of a chart
  status      displays the status of the named release
  template    locally render templates
  test        run tests for a release
  uninstall   uninstall a release
  upgrade     upgrade a release
  verify      verify that a chart at the given path has been signed and is valid
  version     print the client version information

Flags:
      --add-dir-header                   If true, adds the file directory to the header
      --alsologtostderr                  log to standard error as well as files
      --debug                            enable verbose output
  -h, --help                             help for helm
      --kube-context string              name of the kubeconfig context to use
      --kubeconfig string                path to the kubeconfig file
      --log-backtrace-at traceLocation   when logging hits line file:N, emit a stack trace (default :0)
      --log-dir string                   If non-empty, write log files in this directory
      --log-file string                  If non-empty, use this log file
      --log-file-max-size uint           Defines the maximum size a log file can grow to. Unit is megabytes. If the value is 0, the maximum file size is unlimited. (default 1800)
      --logtostderr                      log to standard error instead of files (default true)
  -n, --namespace string                 namespace scope for this request
      --registry-config string           path to the registry config file (default "/home/wojtek/.config/helm/registry.json")
      --repository-cache string          path to the file containing cached repository indexes (default "/home/wojtek/.cache/helm/repository")
      --repository-config string         path to the file containing repository names and URLs (default "/home/wojtek/.config/helm/repositories.yaml")
      --skip-headers                     If true, avoid header prefixes in the log messages
      --skip-log-headers                 If true, avoid headers when opening log files
      --stderrthreshold severity         logs at or above this threshold go to stderr (default 2)
  -v, --v Level                          number for the log level verbosity
      --vmodule moduleSpec               comma-separated list of pattern=N settings for file-filtered logging

Use "helm [command] --help" for more information about a command.
```

## References

* [Helm official docs - Quickstart](https://helm.sh/docs/intro/quickstart/)
* [Helm 3 Example Project](https://github.com/alexellis/helm3-expressjs-tutorial)
* [Helm 3 Example Project - blog post](https://www.civo.com/learn/guide-to-helm-3-with-an-express-js-microservice)
* [Helm Best Practices](https://codefresh.io/docs/docs/new-helm/helm-best-practices/)
* [Package Kubernetes Applications with Helm](https://akomljen.com/package-kubernetes-applications-with-helm/)
* [Drastically Improve your Kubernetes Deployments with Helm](https://itnext.io/drastically-improve-your-kubernetes-deployments-with-helm-5323e7f11ef8)
* [reddit - Deploying multiple similar applications with helm?](https://www.reddit.com/r/kubernetes/comments/8x3znr/deploying_multiple_similar_applications_with_helm/e20vwfc/)
* [reddit - Helm at Reddit: from local dev, staging, to production](https://www.reddit.com/r/kubernetes/comments/80hmaj/helm_at_reddit_from_local_dev_staging_to/)

