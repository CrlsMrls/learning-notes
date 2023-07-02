# Configmaps

- [Configmaps](#configmaps)
  - [Definitions](#definitions)
  - [Describe the ConfigMap](#describe-the-configmap)
    - [List configmaps:](#list-configmaps)
    - [Get information from one:](#get-information-from-one)
    - [Print detailed description with `describe`:](#print-detailed-description-with-describe)
  - [Creation](#creation)
    - [From literal values](#from-literal-values)
    - [From a file](#from-a-file)
    - [From a environment variables file](#from-a-environment-variables-file)
  - [Consuming ConfigMaps in Pods](#consuming-configmaps-in-pods)
    - [ConfigMap as one specific file](#configmap-as-one-specific-file)


## Definitions

ConfigMaps allow to decouple the environment-specific configuration from the containerized applications. 

ConfigMaps store non-confidential data in key-value pairs.

Pods can consume ConfigMaps as:
- environment variables, 
- command-line arguments, or 
- configuration files in a volume.

Aliases: `configmap`, `cm`

## Describe the ConfigMap 

These examples show the output from the basic `get`/`describe` commands:

### List configmaps:

```bash
$  kubectl get configmaps 
NAME               DATA   AGE
kube-root-ca.crt   1      41m
db-config          3      35m
```

### Get information from one:

```
$ kubectl get configmaps db-config -o yaml
apiVersion: v1
data:
  DB_HOST: db.example.com
  DB_NAME: SQL01
  DB_PORT: "3306"
kind: ConfigMap
(...)
```

### Print detailed description with `describe`:

```bash
$ kubectl describe configmaps db-config 
Name:         db-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DB_NAME:
----
SQL01
DB_PORT:
----
3306
DB_HOST:
----
db.example.com
```

## Creation

`kubectl create configmap` creates a config map based on literal values, a file, or directory.

### From literal values

To create ConfigMaps directly from the command line `kubectl create configmap <name> --from-literal=<key>=<value>`.

For example this new ConfigMap with name `webapp-config-map`:

```bash
$ kubectl create configmap webapp-config-map --from-literal=DB_NAME=SQL01
configmap/webapp-config-map created
```

### From a file

To create ConfigMaps from a file `kubectl create configmap <name> --from-file=path/to/bar`.

Example:

```
$ cat nginx.conf
events {
    worker_connections  1024;
}

http {
  server {
    listen 8080;

    location / {
      root   /usr/share/nginx/html;
    }
  }
}

$ kubectl create configmap nginx-config-cm --from-file=./nginx.conf
configmap/nginx-config-cm created
```

This ConfigMap is later used in [ConfigMap as one specific file](#configmap-as-one-specific-file) usage. 

### From a environment variables file

To create ConfigMaps from a environment variables file `kubectl create configmap <name> --from-env-file=path/to/bar`.

Example:

```
$ cat db-conf.env 
  DB_HOST=db.example.com
  DB_NAME=SQL01
  DB_PORT=3306

$ kubectl create configmap db-conf-cm --from-env-file=./db-conf.env 
configmap/db-conf-cm created
```

The keys that are considered invalid will be skipped. The pod will be allowed to start, but the invalid names will be recorded in the event log `InvalidVariableNames`, accessible from `kubectl get events`.

## Consuming ConfigMaps in Pods

### ConfigMap as one specific file

The `volumeMounts.subPath` property specifies a sub-path inside the referenced volume instead of its root.

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    volumeMounts:
      - name: nginx-conf
        mountPath: /etc/nginx/nginx.conf # final destination
        subPath: nginx.conf # the filename in subPath and mountPath are the same
  volumes:
  - name: nginx-conf # Same id used in volumeMounts of the container
    configMap:
       name: nginx-config-cm # ConfigMap id
       items:
         - key: config-nginx.conf  # Filename in the ConfigMap
           path: nginx.conf # Destination filename
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
Notes:
- `spec.volumeMounts.mountPath: /etc/nginx/nginx.conf`: The final destination
- `spec.volumeMounts.subPath: nginx.conf`: The name of the file to be placed inside /etc/nginx, the filename used here and in `mountPath` should be same.
- `spec.volumes.name: nginx-conf`: To access this volume, this name must be used inside `volumeMounts` of the container.
- `spec.volumes.configMap.name: nginx-config-cm`: Name of the configMap.
- `spec.volumes.configMap.key: config-nginx.conf`: Name of the file we had used inside our ConfigMap (under data:).
- `spec.volumes.configMap.path: nginx.conf`: Name of the file to be placed inside `/etc/nginx/` folder. 

Confirm the file is correctly applied in the pod.

```
$ kubectl apply -f pod-nginx.yaml 
pod/nginx created

$ kubectl exec nginx -i -t -- bash
root@nginx:/# cat /etc/nginx/nginx.conf 
events {
    worker_connections  1024;
}

http {
    server {
        listen 8080;
    }
}
```




To modify a pod usage of Configmaps, you must recreate the pod

```bash
$ kubectl get pods webapp-color -o yaml > pod.yaml
$ vim pod.yaml
$ kubectl delete pods webapp-color 
pod "webapp-color" deleted

$ kubectl apply -f pod.yaml 
pod/webapp-color created
```

The content of these can be either a key-value pair:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: green
    image: kodekloud/webapp-color
    imagePullPolicy: Always

(...)
```

Or a reference to a ConfigMap:

```
apiVersion: v1
kind: Pod
metadata:
(...)
spec:
  containers:
  - envFrom:
    - configMapRef:
      name: webapp-config-map
  image: kodekloud/webapp-color
```

