# Labels in Kubernetes  

Labels are key-value pairs that help organize, filter, and select Kubernetes resources efficiently. They are essential for grouping resources, managing deployments, and applying policies.  

## 📌 Basic Usage  

- Use the `-l` flag to **filter** resources based on labels.  
- Show labels for resources using the `--show-labels=true` flag.  
- Combine multiple labels in a query to refine selection.  

### 🔍 Filtering Resources by Label  

Retrieve all resources with a specific label:  

```bash
$ kubectl get all -l env=prod

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
```


### Viewing Labels for Resources

To display labels associated with resources use `--show-labels` flag:

```bash
$ kubectl get pods --show-labels

NAME          READY   STATUS    RESTARTS   AGE     LABELS
app-1-s9pqx   1/1     Running   0          3m10s   bu=finance,env=dev,tier=frontend
auth          1/1     Running   0          3m10s   bu=finance,env=prod
db-2-wv6tn    1/1     Running   0          3m10s   bu=finance,env=prod,tier=db
app-1-2zwtk   1/1     Running   0          3m10s   bu=finance,env=dev,tier=frontend
app-1-zzxdf   1/1     Running   0          3m9s    bu=finance,env=prod,tier=frontend
app-1-xjnpg   1/1     Running   0          3m10s   bu=finance,env=dev,tier=frontend
```

### 🎯 Selecting Resources with Multiple Labels

Filter resources using multiple labels to get more precise results.

```bash
$ kubectl get pods -l bu=finance,env=prod,tier=frontend
NAME          READY   STATUS    RESTARTS   AGE
app-1-zzxdf   1/1     Running   0          3m20s
```

### 🏷️ Applying Labels to Resources

#### Adding Labels
You can add labels to existing resources using the `kubectl label` command:

```bash
$ kubectl label pod my-pod env=production
```

#### Modifying Labels
To modify an existing label:

```bash
$ kubectl label pod my-pod env=staging --overwrite
$ kubectl get pod my-pod --show-labels
NAME     READY   STATUS    RESTARTS   AGE   LABELS
my-pod   1/1     Running   0          1m    env=staging
```

#### To remove a label:

```bash
$ kubectl label pod my-pod env-
$ kubectl get pod my-pod --show-labels
NAME     READY   STATUS    RESTARTS   AGE   LABELS
my-pod   1/1     Running   0          1m    <none>
```
