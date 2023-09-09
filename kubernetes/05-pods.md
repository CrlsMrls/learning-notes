# Pods

At the core of Kubernetes is the Pod. Pods represent and hold a collection of one or more containers. Generally, if you have multiple containers with a hard dependency on each other, you package the containers inside a single pod.

Pods also have Volumes. Volumes are data disks that live as long as the pods live, and can be used by the containers in that pod. Pods provide a shared namespace for their contents which means that the two containers inside of our example pod can communicate with each other, and they also share the attached volumes.

Pods also share a network namespace. This means that there is one IP Address per pod.

- [Pods](#pods)
  - [Pod lifecycle](#pod-lifecycle)
  - [Pod creation](#pod-creation)
  - [Multi-container pods](#multi-container-pods)


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

Although when possible, it is better to always use **apply**. Modifying an already created resource from a definition file requires **apply** instead of reusing **create**.

```bash
$ kubectl create -f definition.yml
Error from server (AlreadyExists): error when creating "definition.yml": pods "front-ui" already exists

$ kubectl apply -f definition.yml
pod/front-ui configured
```

## Multi-container pods

Multi-container patterns help the main container, these are the most common patterns: 
- A **sidecar** container performs some task that helps the main container.
- An **ambassador** container proxies network traffic to and/or from the main container.
- An **adapter** container transforms the main container's output.

https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/

**Init containers** could be also considered as multi-container pods, with the particularity that they do not run in parallel: init containers stop working once they finish their execution.

Multi-container pods allow coupled containers to share resources. All containers in the Pod:
- can access the shared volumes, allowing those containers to share data. 
- share the network namespace, including the IP address and network ports. Inside a Pod, the containers can communicate with one another using `localhost`. 
- can share the standard inter-process communications (IPC)


