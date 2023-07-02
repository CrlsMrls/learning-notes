
# Taints

Kubernetes taints are used to repel pods from nodes. Taints are applied to nodes and tolerations are applied to pods.

```bash
controlplane ~ ➜  kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   17m   v1.24.0
node01         Ready    <none>          17m   v1.24.0

kubectl taint nodes node01 spray=mortein:NoSchedule
node/node01 tainted

kubectl describe node node01
Name:               node01
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node01
                    kubernetes.io/os=linux
(...)
Taints:             spray=mortein:NoSchedule
(...)
```

If we want to run a pod the pod won't start and stay in `Pending` state

```bash
kubectl run nginx --image=nginx
kubectl get pod nginx 
NAME       READY   STATUS    RESTARTS   AGE
nginx   0/1     Pending   0          80s


kubectl describe pod nginx 
(...)
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  95s   default-scheduler  0/2 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 1 node(s) had untolerated taint {spray: mortein}. preemption: 0/2 nodes are available: 2 Preemption is not helpful for scheduling.
```


Pod cannot tolerate taint `mortein`, the state says: `1 node(s) had untolerated taint {spray: mortein}`


Actually, this is how controlplane node is protected from running pods:

```bash
kubectl describe node controlplane
Name:               controlplane
Roles:              control-plane
(...)
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
Unschedulable:      false
```

To remove the taint on controlplane:

```bash
k taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-
node/controlplane untainted
```

Now the node controlplane can accept any pod.

```bash
kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
bee        1/1     Running   0          6m30s   10.244.1.2   node01         <none>           <none>
mosquito   1/1     Running   0          14m     10.244.0.4   controlplane   <none>           <none>
```

## Tolerations in pod definition

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
    resources: {}
  tolerations:
  - key: "spray"
    operator: "Equal"
    value: "mortein"
    effect: "NoSchedule"
```
