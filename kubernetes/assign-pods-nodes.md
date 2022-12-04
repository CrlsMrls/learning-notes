# Assigning Pods to Nodes

The scheduler watches for newly created pods with no assigned node, and selects a node for them to run on. These strategies define this process:

- [Node Selector](#node-selector)
- [Affinity](#affinity)
- [Taints](#taints)

## Node Selector

A pod can be attached directly to a node using the `spec.nodeSelector` in the Pod description file. This field then matches against node labels.

The following example attaches a label to the node and then creates a pod to run in that node.
```
$ kubectl label node node01 disktype=ssd 
node/node01 labeled

$ cat pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  nodeSelector:
    disktype: ssd

$ kubectl apply -f pod.yml
pod/nginx created

$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP           NODE 
nginx-6d7486c96b-2zs7d   1/1     Running   0          5m12s   10.244.1.4   node01 
```

If the pod specifies a `nodeSelector` but there is no node with that label, the pod will stay in a `Pending` state. As in the following example:

```
$ kubectl describe pod nginx 
Name:             nginx
Namespace:        default
(...)
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  46s   default-scheduler  0/1 nodes are available: 1 node(s) didn't match Pod's node affinity/selector. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
```

## Affinity

Node affinity is conceptually similar to `nodeSelector`, allowing more complex rules.

You can use the operator field to specify a logical operator for Kubernetes to use when interpreting the rules. You can use `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt` and `Lt`.

Node affinities are placed in the `.spec.affinity.nodeAffinity` field from the Pod spec.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: blue
    spec:
      containers:
      - image: nginx
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: disktype
                operator: In
                values:
                - ssd 
```

This manifest describes a deployment that has a `preferredDuringSchedulingIgnoredDuringExecution` node affinity, `disktype: ssd`. This means that the pod will prefer a node that has a `disktype=ssd` label.

To apply a label to a node use the `kubectl label node` command: 

```
$ kubectl label node node01 disktype=ssd 
node/node01 labeled

$ kubectl apply -f deployment.yml

$ kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE 
blue-6d7486c96b-2zs7d   1/1     Running   0          5m12s   10.244.1.4   node01 
blue-6d7486c96b-chxlx   1/1     Running   0          5m9s    10.244.1.6   node01
blue-6d7486c96b-rqs5z   1/1     Running   0          5m11s   10.244.1.5   node01
```

Node affinity can be used to place pods in separate nodes or to co-locate Pods from different deployments. Check the [documentation for a full example](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)

## Taints

If affinity tells the pod to go to a pod, taints offer a different approach: they allow to repel pods. 

*Taints* allow node to repel a set of pods. When a taint is applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

*Tolerations* are applied to pods. When the correct Toleratints match the taint, the pod can be scheduled in the node (although does not guarantee the scheduling).

There can be multiple taints on the same node and multiple tolerations on the same pod.

### Use case for taints
Example, a node has a GPU, we want to protect this node from running pods that are not GPU specific.

```
$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   17m   v1.24.0
node01         Ready    <none>          17m   v1.24.0
```

To apply a taint key = `type` and value = `gpu`:
```
$ kubectl taint nodes node01 type=gpu:NoSchedule
node/node01 tainted
```

Check taint is correctly applied:

```
$ kubectl describe node node01
Name:               node01
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
(...)
Taints:             type=gpu:NoSchedule
(...)
```

If we want to run a pod the pod won't start and stay in `Pending` state

```
$ kubectl run nginx --image=nginx

$ kubectl get pod nginx 
NAME       READY   STATUS    RESTARTS   AGE
nginx   0/1     Pending   0          80s

$ kubectl describe pod nginx 
(...)
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  95s   default-scheduler  0/2 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 1 node(s) had untolerated taint {type: gpu}. preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.
```

Pod cannot tolerate taint `type:gpu`.

### Create Tolerations

Following the previous example, to create a pod with the valid toleration, add to the `spec`:

```
apiVersion: v1
kind: Pod
metadata:
  name: ml
spec:
  containers:
  - image: tensorflow/tensorflow
    name: ml
    resources: {}
  tolerations:
  - key: "type"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

### Remove taints 

This is how controlplane node is protected from running pods:

```
$ kubectl describe node controlplane 
Name:               controlplane
Roles:              control-plane
(...)
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Unschedulable:      false
```

To remove the taint on controlplane:

```
$ kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
node/controlplane untainted
```

Now the node controlplane can accept any pod.

### Type of Taints

- `NoSchedule`: Kubernetes will not schedule the pod onto that node
- `NoExecute`: the pod will be evicted from the node (if it is already running on the node), and will not be scheduled onto the node (if it is not yet running on the node).


