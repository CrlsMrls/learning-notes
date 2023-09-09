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