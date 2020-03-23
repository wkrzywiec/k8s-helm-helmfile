# Kubernetes Deployment

### 0. Install Minikube & Kubectl

### 1. Start a new Minikube cluster

In order to run a minikube cluster:
```bash
$ minikube start
```

To check the status of the cluster:
```bash
$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

To check that `kubectl` is properly configured check this:
```bash
$ kubectl cluster-infor
Kubernetes master is running at https://127.0.0.1:32768
KubeDNS is running at https://127.0.0.1:32768/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

### 2. Add `postgres` 

First create `postgres-pvc.yaml` file. And then apply it:
```bash
$ kubectl apply -f postgres-pvc.yaml
persistentvolumeclaim/postgres-persistent-volume-claim created

$ kubectl get pvc
NAME                               STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-persistent-volume-claim   Bound    pvc-f5d9b781-9cdf-4a4c-8c9b-2edb8330d139   4Gi        RWO            standard       10s
```

Then create a new file - `postgres-deployment.yaml` and then apply it:
```bash
$ kubectl apply -f postgres-deployment.yaml
deployment.apps/postgres created

$ 

```

https://github.com/wkrzywiec/kanban-board/blob/master/docker-compose.yml

Add Volume, Secret, Deployment
https://kubernetes.io/docs/concepts/storage/persistent-volumes/

### 3. Add `kanban-app` Deployment & Service 

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

### References

https://kubernetes.io/docs/reference/kubectl/overview/
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
