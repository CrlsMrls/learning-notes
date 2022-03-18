# Pods

At the core of Kubernetes is the Pod. Pods represent and hold a collection of one or more containers. Generally, if you have multiple containers with a hard dependency on each other, you package the containers inside a single pod.

Pods also have Volumes. Volumes are data disks that live as long as the pods live, and can be used by the containers in that pod. Pods provide a shared namespace for their contents which means that the two containers inside of our example pod can communicate with each other, and they also share the attached volumes.

Pods also share a network namespace. This means that there is one IP Address per pod.

## Get information on pods

```bash
$ kubectl get nodes
NAME           STATUS   ROLES                  AGE     VERSION
controlplane   Ready    control-plane,master   4m25s   v1.22.2+k3s2
```

For getting environment and running OS:

```bash
$ kubectl get nodes -o wide
NAME           STATUS   ROLES                  AGE     VERSION        INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
controlplane   Ready    control-plane,master   6m28s   v1.22.2+k3s2   172.25.0.48   <none>        Alpine Linux v3.14   5.4.0-1028-gcp   containerd://1.5.7-k3s1
```

## Pod lifecycle

Create a new pod with the `nginx` image.

```bash
$ kubectl run nginx --image=nginx
pod/nginx created
```

Get information on running pods

```bash
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          93s
```

Getting detailed information about a pod

```bash
$ kubectl describe pods <pod-id>
Name:         nginx
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.83
## .....
```

Finally, delete the pod

```bash
$ kubectl delete pod <pod-id>
pod "<pod-id>" deleted
```

## Pod creation

Create a manifest file with kubectl using the options **--dry-run=client -o yaml**

```bash
$ kubectl run front-ui --image=nginx --labels="tier=frontend,app=angular" --dry-run=client -o yaml > definition.yml

$ cat definition.yml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    app: angular
    tier: frontend
  name: front-ui
spec:
  containers:
  - image: nginx
    name: front-ui
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Create a resource from a manifest file with the **kubectl create -f** command.

```bash
$ kubectl create -f definition.yml
pod/front-ui created
```

Although it is better to always use **apply**. Modifying an already created resource from a definition file requires **apply** instead of reusing **create**.

```bash
$ kubectl create -f definition.yml
Error from server (AlreadyExists): error when creating "definition.yml": pods "front-ui" already exists

$ kubectl apply -f definition.yml
pod/front-ui configured
```
