# Labels

Labels are very useful for filtering and selecting resources.

- Filter results using labels with the `-l` flag.
- Show labels with the `--show-labels=true`.
- Use both commands together.
- Use many labels with the `-l` flag.

```bash
$ kubectl get all -l=env=prod
NAME              READY   STATUS    RESTARTS   AGE
pod/auth          1/1     Running   0          2m34s
pod/db-2-wv6tn    1/1     Running   0          2m34s
pod/app-1-zzxdf   1/1     Running   0          2m33s
pod/app-2-s2j2h   1/1     Running   0          2m34s

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/app-1   ClusterIP   10.43.240.152   <none>        3306/TCP   2m33s

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/db-2    1         1         1       2m34s
replicaset.apps/app-2   1         1         1       2m34s

$ kubectl get pods -l=bu=finance --show-labels=true
NAME          READY   STATUS    RESTARTS   AGE     LABELS
app-1-s9pqx   1/1     Running   0          3m10s   bu=finance,env=dev,tier=frontend
auth          1/1     Running   0          3m10s   bu=finance,env=prod
db-2-wv6tn    1/1     Running   0          3m10s   bu=finance,env=prod,tier=db
app-1-2zwtk   1/1     Running   0          3m10s   bu=finance,env=dev,tier=frontend
app-1-zzxdf   1/1     Running   0          3m9s    bu=finance,env=prod,tier=frontend
app-1-xjnpg   1/1     Running   0          3m10s   bu=finance,env=dev,tier=frontend

$ kubectl get pod -l=bu=finance,env=prod,tier=frontend
NAME          READY   STATUS    RESTARTS   AGE
app-1-zzxdf   1/1     Running   0          3m20s
```

# Updating a Deployment

## Rollout creation

Create a deployment and check the rollout status:

```bash
$ kubectl create deployment nginx --image=nginx:1.21
deployment.apps/nginx created
 
$ kubectl rollout status deployment nginx
Waiting for deployment "nginx" rollout to finish: 0 of 1 updated replicas are available...
deployment "nginx" successfully rolled out
```

## Check history

First, check the history of the deployment, then a specific rollout version with `--revision` flag.

```bash
$ kubectl rollout history deployment nginx
deployment.extensions/nginx
REVISION CHANGE-CAUSE
1     <none>

$ kubectl rollout history deployment nginx --revision=1
deployment.extensions/nginx with revision #1
 
Pod Template:
 Labels:    app=nginx    pod-template-hash=6454457cdb
 Containers:  nginx:  Image:   nginx:1.21
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>
```

## Record the changes

Save the command used to create/update a deployment with the `--record` flag. This will fill the `change-cause` field in the rollout history:

```
$ kubectl set image deployment nginx nginx=nginx:1.22 --record
deployment.extensions/nginx image updated

$ kubectl rollout history deployment nginx
deployment.extensions/nginx
 
REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment nginx nginx=nginx:1.22 --record=true

$ kubectl edit deployments. nginx --record
deployment.extensions/nginx edited
 
$ kubectl rollout history deployment nginx
REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment nginx nginx=nginx:1.17 --record=true
3     kubectl edit deployments. nginx --record=true
```

## Rollback - Undoing the changes

Rollback to a specific revision with the `--to-revision` flag.

```bash
$ kubectl rollout undo deployment nginx --to-revision=1
deployment.apps/nginx rolled back

$ kubectl describe deployment nginx | grep -i image:
Image: nginx:1.21
```

## Identify the strategy

Possible values:
- `"Recreate"` Kill all existing pods before creating new ones.
- `"RollingUpdate"` gradually scale down the old ReplicaSets and scale up the new one.

```
$ k describe deploy frontend
Name:                   frontend
(...)
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
(...)
```

Change the rollout strategy by modifying the deployment specification to `.spec.strategy.type: Recreate`.

```
$ k describe deploy frontend
Name:               frontend
(...)
Replicas:           4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:       Recreate
(...)
```