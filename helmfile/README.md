# Helmfile Deployment

### Table of content
* [Install Helm](#Install-Helm)
* [Blueprint Helm Charts](#Blueprint-Helm-Charts)
  * [App Helm Chart](#App-Helm-Chart)
  * [Postgres Helm chart](#Postgres-Helm-chart)
  * [Ingress Helm Chart](#Ingress-Helm-Chart)
* [Override Values.yaml for each Helm release](#Override-Values.yaml-for-each-Helm-release)
* [Create Helm releases](#Create-Helm-releases)
* [Maintanance](#Maintanance)
* [References](#References)


## Install Helm
Installing Helm - https://helm.sh/docs/intro/install/
Installing Helmfile - https://github.com/roboll/helmfile#installation

In order to install Helmfile first download the executable. For Ubuntu (Linux) first move it to the `/usr/local/bin` folder and change the permissions:

```bash
$ sudo cp ./helmfile /usr/local/bin
$ sudo chmod +x ./helmfile
```

After that you should be able to run the command:

```bash
$ helmfile --version
helmfile version v0.108.0
```

## Blueprint Helm Charts

Within this project there are 3 types of Helm charts, which will be used to deploy all Kubernetes objects. They are located in `./charts` folder.

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

### Ingress Helm Chart

And the last chart, is responsible for creating and configurating Ingress Controller. 

It creates the default backend service - marked as `dependency` in `Chart.yaml`:

```yaml
dependencies:
  - name: nginx-ingress
    version: 1.36.0
    repository: https://kubernetes-charts.storage.googleapis.com/
```

And the configuration of Ingress Controller, which is made by combining of `/templates/ingress.yaml` and `values.yaml`. 

## Override Values.yaml for each Helm release

To create each Helm release, i.e. deploy each application, relevant YAML file was created and put in `./values.yaml`. E.g. `adminer.yaml` contains specific values, like base Docker image or name for Deployment.

In the root folder (`./helmfile`) there is a `helmfile.yaml` which combines both charts and values into Helm releases. It's a master file that defines entire environment with all apps inside of it. 

## Create Helm releases

First you need to add repository defined in `helmfile.yaml`:

```bash
$ helmfile repos
Adding repo stable https://kubernetes-charts.storage.googleapis.com
"stable" has been added to your repositories

Updating repo
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 
```

Then run `helm sync` command to deploy all apps (Helm releases defined in `helmfile.yaml`):

```bash
$ helmfile sync
Adding repo stable https://kubernetes-charts.storage.googleapis.com
"stable" has been added to your repositories

Updating repo
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 

Building dependency release=postgres, chart=charts/postgres
Building dependency release=adminer, chart=charts/app
Building dependency release=kanban-app, chart=charts/app
Building dependency release=kanban-ui, chart=charts/app
Building dependency release=ingress, chart=charts/ingress
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading nginx-ingress from repo https://kubernetes-charts.storage.googleapis.com/
Deleting outdated charts

Affected releases are:
  adminer (./charts/app) UPDATED
  ingress (./charts/ingress) UPDATED
  kanban-app (./charts/app) UPDATED
  kanban-ui (./charts/app) UPDATED
  postgres (./charts/postgres) UPDATED

Upgrading release=postgres, chart=charts/postgres
Upgrading release=adminer, chart=charts/app
Upgrading release=kanban-ui, chart=charts/app
Upgrading release=kanban-app, chart=charts/app
Upgrading release=ingress, chart=charts/ingress
Release "kanban-ui" does not exist. Installing it now.
NAME: kanban-ui
LAST DEPLOYED: Sat Apr 11 19:39:41 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

Listing releases matching ^kanban-ui$
kanban-ui	default  	1       	2020-04-11 19:39:41.082768248 +0200 CEST	deployed	app-0.1.0	1.16.0     

Release "kanban-app" does not exist. Installing it now.
NAME: kanban-app
LAST DEPLOYED: Sat Apr 11 19:39:41 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

Listing releases matching ^kanban-app$
Release "adminer" does not exist. Installing it now.
NAME: adminer
LAST DEPLOYED: Sat Apr 11 19:39:41 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

Listing releases matching ^adminer$
kanban-app	default  	1       	2020-04-11 19:39:41.115007764 +0200 CEST	deployed	app-0.1.0	1.16.0     

Release "postgres" does not exist. Installing it now.
NAME: postgres
LAST DEPLOYED: Sat Apr 11 19:39:41 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

Listing releases matching ^postgres$
adminer	default  	1       	2020-04-11 19:39:41.096703774 +0200 CEST	deployed	app-0.1.0	1.16.0     

postgres	default  	1       	2020-04-11 19:39:41.104092113 +0200 CEST	deployed	postgres-0.1.0	1.16.0     

Release "ingress" does not exist. Installing it now.
NAME: ingress
LAST DEPLOYED: Sat Apr 11 19:39:41 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

Listing releases matching ^ingress$
ingress	default  	1       	2020-04-11 19:39:41.11613135 +0200 CEST	deployed	ingress-0.1.0	1.16.0     


UPDATED RELEASES:
NAME         CHART               VERSION
kanban-ui    ./charts/app          0.1.0
kanban-app   ./charts/app          0.1.0
adminer      ./charts/app          0.1.0
postgres     ./charts/postgres     0.1.0
ingress      ./charts/ingress      0.1.0
```

## Maintanance

To get help about helmfile:
```bash
$ helmfile help
NAME:
   helmfile

USAGE:
   helmfile [global options] command [command options] [arguments...]

VERSION:
   v0.108.0

COMMANDS:
     deps      update charts based on the contents of requirements.yaml
     repos     sync repositories from state file (helm repo add && helm repo update)
     charts    DEPRECATED: sync releases from state file (helm upgrade --install)
     diff      diff releases from state file against env (helm diff)
     template  template releases from state file against env (helm template)
     lint      lint charts from state file (helm lint)
     sync      sync all resources from state file (repos, releases and chart deps)
     apply     apply all resources from state file only when there are changes
     status    retrieve status of releases in state file
     delete    DEPRECATED: delete releases from state file (helm delete)
     destroy   deletes and then purges releases
     test      test releases from state file (helm test)
     build     output compiled helmfile state(s) as YAML
     list      list releases defined in state file
     help, h   Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --helm-binary value, -b value           path to helm binary (default: "helm")
   --file helmfile.yaml, -f helmfile.yaml  load config from file or directory. defaults to helmfile.yaml or `helmfile.d`(means `helmfile.d/*.yaml`) in this preference
   --environment default, -e default       specify the environment name. defaults to default
   --state-values-set value                set state values on the command line (can specify multiple or separate values with commas: key1=val1,key2=val2)
   --state-values-file value               specify state values in a YAML file
   --quiet, -q                             Silence output. Equivalent to log-level warn
   --kube-context value                    Set kubectl context. Uses current context by default
   --no-color                              Output without color
   --log-level value                       Set log level, default info
   --namespace value, -n value             Set namespace. Uses the namespace set in the context by default, and is available in templates as {{ .Namespace }}
   --selector value, -l value              Only run using the releases that match labels. Labels can take the form of foo=bar or foo!=bar.
                                           A release must match all labels in a group in order to be used. Multiple groups can be specified at once.
                                           --selector tier=frontend,tier!=proxy --selector tier=backend. Will match all frontend, non-proxy releases AND all backend releases.
                                           The name of a release can be used as a label. --selector name=myrelease
   --allow-no-matching-release             Do not exit with an error code if the provided selector has no matching releases.
   --interactive, -i                       Request confirmation before attempting to modify clusters
   --help, -h                              show help
   --version, -v                           print the version
```

## References

https://github.com/roboll/helmfile

