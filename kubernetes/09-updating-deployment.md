# Updating deployments

Deployments support updating images to a new version through a rolling update mechanism. When a Deployment is updated with a new version, it creates a new ReplicaSet and slowly increases the number of replicas in the new ReplicaSet as it decreases the replicas in the old ReplicaSet.

Kubernetes triggers a Deployment's rollout when the Pod template (`.spec.template`) is changed. This will begin a rolling update. 

- [Updating deployments](#updating-deployments)
  - [1. Rollout status](#1-rollout-status)
  - [2. Rollout history](#2-rollout-history)
  - [3. Rollout pause](#3-rollout-pause)
  - [4. Rollout resume](#4-rollout-resume)
  - [5. Rollout undo](#5-rollout-undo)
  - [6. Annotate the rollout](#6-annotate-the-rollout)


## 1. Rollout status
Verify the **current state** of the rollout:

```bash
$ kubectl rollout status deployment/frontend
Waiting for deployment "frontend" rollout to finish: 0 of 1 updated replicas are available...
deployment "frontend" successfully rolled out

Waiting for deployment "frontend" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "frontend" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "frontend" rollout to finish: 3 of 5 updated replicas are available...
Waiting for deployment "frontend" rollout to finish: 4 of 5 updated replicas are available...
deployment "frontend" successfully rolled out
```

During this period, check for `kubectl get deployments`, `kubectl get pods` and `kubectl get rs`

## 2. Rollout history

To check the history of the deployment, and then a specific rollout version with `--revision` flag.

```bash
$ kubectl rollout history deployment frontend
deployment.extensions/frontend
REVISION CHANGE-CAUSE
1     <none>

$ kubectl rollout history deployment frontend --revision 2
deployment.apps/frontend with revision #2
Pod Template:
  Labels:       app=web
        pod-template-hash=5fb69bcb69
  Annotations:  kubernetes.io/change-cause: kubectl apply --filename=web2.yaml --record=true
  Containers:
   nginx:
    Image:      nginx:1.24-alpine
    Port:       80/TCP
    Host Port:  0/TCP
  ...
```

## 3. Rollout pause
If you detect problems, **pause the running rollout** to stop the update:

```bash
$ kubectl rollout pause deployment/frontend
```

## 4. Rollout resume
The rollout is paused which means that some pods are at the new version and some pods are at the older version. We can **continue the rollout** using the resume command.

```bash
$ kubectl rollout resume deployment/frontend
```

## 5. Rollout undo
If a bug was detected in the new version, use the rollout command to **roll back** to the previous version:

```bash
$ kubectl rollout undo deployment/frontend
```

Rollback to a specific revision with the `--to-revision` flag.

```bash
$ kubectl rollout undo deployment nginx --to-revision=1
deployment.apps/nginx rolled back

$ kubectl describe deployment nginx | grep -i image:
Image: nginx:1.21
```

## 6. Annotate the rollout

The Deployment annotation `kubernetes.io/change-cause` value is copied to `CHANGE-CAUSE` column of `rollout history`:

```bash
$ kubectl annotate deployment frontend kubernetes.io/change-cause="image updated to 1.24"
deployment.apps/frontend annotated

$ kubectl rollout history deployment frontend 
deployment.apps/frontend 
REVISION  CHANGE-CAUSE
2         kubectl apply --filename=web2.yaml --record=true
3         image updated to 1.24
```