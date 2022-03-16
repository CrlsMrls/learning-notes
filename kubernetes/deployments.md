# Deployments

A Deployment provides declarative updates for Pods and ReplicaSets. They provide the mechanisms to roll updates and rollbacks.

## Show deployment information

Show basic summary for deployments **kubectl get deployment** or **kubectl get deploy**

```bash
$  kubectl get deploy
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
frontend-deployment   0/4     4            0           6s
```

## Deployment definition file

Deployment definition files look very similar to [ReplicaSets](./replica-sets.md). **matchLabels** must match the label speficied for the pod inside the template.

```bash
$  cat ./deployment-definition.yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      name: frontend-pod
  template:
    metadata:
      labels:
        name: frontend-pod
    spec:
      containers:
      - name: httpd
        image: httpd:2.4-alpine


$ kubectl apply -f ./deployment-def.yml
deployment.apps/httpd-frontend created

```

## Rollout strategy

Find out the rollout strategy by describing the deployment. In the following example it is **RollingUpdate**

```bash
$ kubectl describe deployment
Name:                   frontend
Namespace:              default
CreationTimestamp:      Thu, 23 Dec 2021 16:40:09 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=webapp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate

```

With **RollingUpdate** strategy, PODs are upgraded few at a time

all PODs are taken down before upgrading any.

Find out what is happening in the current deployment rollout by showing the **status**:

```bash
$ kubectl rollout status deployment frontend
deployment "frontend" successfully rolled out
```

Rollback a deployment with **undo** command:

```bash
$ kubectl rollout undo deployment frontend
deployment.apps/frontend rolled back
```

Get the history of all rollouts

```bash
$ kubectl rollout history deployment frontend
```
