# Replica sets

Replica sets guarantee a number of identical Pods are running. They handle restarting Pods if they happen to go down for some reason.

Deployments are normally used instead of Replica Sets, as they provide more features and flexibility. Deployments use Replica Sets behind the scenes.

- [Replica sets](#replica-sets)
  - [Definition file](#definition-file)
  - [Getting information](#getting-information)
  - [Scale up and down](#scale-up-and-down)

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
As with other objects, the **kubectl create** command can be used to create a ReplicaSet. The **--dry-run=client** option is used to prevent the ReplicaSet from being created, use it together with `-o yaml` to output the manifest file.

The **kubectl apply** command can be used to create a ReplicaSet from a file.

In the following example the ReplicaSet will create 4 replicas of the nginx image, from the `replicaset-definition.yaml` file.

```yaml
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
```

```bash
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
