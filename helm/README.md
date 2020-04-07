# Helm Deployment


## Install Minikube & Kubectl


Installing Docker - https://docs.docker.com/install/
Installing minikube - https://kubernetes.io/docs/tasks/tools/install-minikube/
Installing kubectl - https://kubernetes.io/docs/tasks/tools/install-kubectl/
Installing Helm - https://helm.sh/docs/intro/install/

## Start a new Minikube cluster

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

## Preconfiguration
## Edit hosts file


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


### Helm config

Add repository with official stable Helm charts:

```bash
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
"stable" has been added to your repositories
```

Add repository with Adminer:
```bash
helm repo add cetic https://cetic.github.io/helm-charts
"cetic" has been added to your repositories
```


### Helm Umbrella Chart

First create an Umbrella Chart:
```bash
$ helm create helm
Creating helm

$ tree -a
.
├── charts
├── Chart.yaml                      # A YAML file containing information about the chart
├── .helmignore
├── README.md
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml                     # The default configuration values for this chart
```

Chart structure - https://helm.sh/docs/topics/charts/



### Adminer

helm install adminer cetic/adminer --version 0.1.3

Ingress
=======

Install using Helm, go to official docs - https://kubernetes.github.io/ingress-nginx/deploy/#using-helm

```bash
$ helm install my-nginx stable/nginx-ingress
NAME: my-nginx
LAST DEPLOYED: Wed Apr  1 07:42:46 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The nginx-ingress controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w my-nginx-nginx-ingress-controller'

An example Ingress that makes use of the controller:

  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls

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

