# Deployments

A Deployment provides declarative updates for Pods and ReplicaSets. They provide the mechanisms to roll updates and rollbacks.

The main benefit of Deployments is in abstracting away the low level details of managing Pods. Behind the scenes Deployments use [Replica Sets](replica-sets.md) to manage starting and stopping the Pods. If Pods need to be updated or scaled, the Deployment will handle that.

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

## Rollup and rollback updates

Deployments support updating images to a new version through a rolling update mechanism. When a Deployment is updated with a new version, it creates a new ReplicaSet and slowly increases the number of replicas in the new ReplicaSet as it decreases the replicas in the old ReplicaSet

To update your Deployment, run the following command:

```
kubectl edit deployment hello
```

Change the image version in the containers section of the Deployment:

```
...
containers:
- name: hello
  image: whatever/image:2.0.0
...
```

This will update the Deployment to your cluster and Kubernetes will begin a rolling update. To see the** rollout history**:

```
kubectl rollout history deployment/frontend
```

If you detect problems, **pause the running rollout** to stop the update:

```
kubectl rollout pause deployment/frontend
```

Verify the **current state** of the rollout:

```
kubectl rollout status deployment/frontend
```

The rollout is paused which means that some pods are at the new version and some pods are at the older version. We can **continue the rollout** using the resume command.

```
kubectl rollout resume deployment/frontend
```

If a bug was detected in the new version, use the rollout command to **roll back** to the previous version:

```
kubectl rollout undo deployment/frontend
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

## Canary deployments

When you want to test a new deployment in production with a subset of your users, use a canary deployment. Canary deployments allow you to release a change to a small subset of your users to mitigate risk associated with new releases.

If you want to ensure that a user didn't get served by the Canary deployment, each user should "stick" to one deployment or the other.

You can do this by creating a service with session affinity. This way the same user will always be served from the same version. Add a new `sessionAffinity` field and set to `ClientIP`. All clients with the same IP address will have their requests sent to the same version of the hello application.

```
kind: Service
apiVersion: v1
metadata:
  name: "hello"
spec:
  sessionAffinity: ClientIP
  ...
```

## Blue-green deployments

Rolling updates are ideal because they allow you to deploy an application slowly with minimal overhead, minimal performance impact, and minimal downtime. There are instances where it is beneficial to modify the load balancers to point to that new version only after it has been fully deployed. In this case, blue-green deployments are the way to go.

Kubernetes achieves this by creating two separate deployments; one for the **old "blue"** version and one for the **new "green"** version. Once the new "green" version is up and running, you'll switch over to using that version by updating the Service, which will act as the router.

A major downside of blue-green deployments is that you will need to have at least 2x the resources in your cluster necessary to host your application.
