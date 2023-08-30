# Resource limits

```
template:
   metadata:
     labels:
       app: royal-jelly
   spec:
     containers:
     - image: nginx:stable
       name: nginx
       ports:
       - containerPort: 80
         protocol: TCP
       resources:
         limits:
           cpu: 500m
           memory: 256Mi
         requests:
           cpu: 250m
           memory: 128Mi
```

The ResourceQuota admission controller must be enabled in the environment, this is done by editing the `/etc/kubernetes/manifests/kube-apiserver.yaml` file and passing the command `- --enable-admission-plugins=NodeRestriction,ResourceQuota` to the container.

```
$ cat resourcequota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: enable-limits
  namespace: hive
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi

$ k apply -f resourcequota.yaml
resourcequota/enable-limits created

$ kubectl get resourcequota -n hive
NAME            AGE   REQUEST                                            LIMIT
enable-limits   12s   requests.cpu: 750m/1, requests.memory: 384Mi/1Gi   limits.cpu: 1500m/2, limits.memory: 768Mi/2Gi
```