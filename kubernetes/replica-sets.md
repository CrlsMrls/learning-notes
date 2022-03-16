# Replica sets

Replica sets guarantee a number of identical Pods are running. They handle restarting Pods if they happen to go down for some reason.

## Definition file

Describe how to create a Replica set

```bash
$ kubectl explain replicaset
KIND:     ReplicaSet
VERSION:  apps/v1

DESCRIPTION:
     ReplicaSet ensures that a specified number of pod replicas are running at
     any given time.
## ...
```

These two commands create a new replica set:

```bash
$ cat ./replicaset-definition.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: new-replica-set
spec:
  replicas: 4
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx

$ kubectl apply -f ./replicaset-definition.yaml
replicaset.apps/new-replica-set created
```

## Getting information

Once the deployment is created, Kubernetes will create a ReplicaSet for the Deployment.

Get information on the ReplicaSets with **kubectl get replicaset** or **kubectl get rs**:

```bash
$ kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       5s

$ kubectl get rs -o wide
NAME              DESIRED   CURRENT   READY   AGE   CONTAINERS          IMAGES       SELECTOR
new-replica-set   4         4         0       30s   nginx               nginx        tier=frontend
```

Getting even more information with describe

```bash
$ kubectl describe replicaset new-replica-set
Name:         new-replica-set
Namespace:    default
Selector:     tier=frontend
## ....

```

## Scale up and down

Scaling up/down replicas can be done directly, without modifying the specification file. Although this is not recommended as it does not satisfies infrastructure as code.

```bash
$ kubectl scale rs new-replica-set --replicas=5

```
