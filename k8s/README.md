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

Next create a **ConfigMap** that will hold all database credentials - `postgres-configmap.yaml`:
```bash
$ kubectl apply -f postgres-configmap.yaml

$ kubectl get configmaps
NAME              DATA   AGE
postgres-config   3      11s
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

### Maintenance

Official **kubectle** cheatsheet:

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

In order to **delete** a Kubernetes resource (Pod, Deployemnt, etc.) use command:
```bash
$ kubectl delete -f ./k8s/postgres-deployment.yaml
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

### References

https://severalnines.com/database-blog/using-kubernetes-deploy-postgresql

https://kubernetes.io/docs/reference/kubectl/overview/
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
