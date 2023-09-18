# Configmaps

ConfigMaps allow to decouple the environment-specific configuration from the containerized applications. 

- The data stored in a ConfigMap cannot exceed 1 MiB.
- ConfigMaps store non-confidential data for other objects to use.
- Unlike most Kubernetes objects that have a `spec`, a ConfigMap has `data` and `binaryData` fields: 
  - The `data` field is used to store key-value pairs in UTF-8 strings, and
  - The `binaryData` field is used to store binary data in base64-encoded strings.
- Pods can consume ConfigMaps (in the same namespace) in several ways, including:
  - environment variables, 
  - command-line arguments, or 
  - configuration files in a volume.

Aliases: `configmap`, `cm`

- [Configmaps](#configmaps)
  - [Describe the ConfigMap](#describe-the-configmap)
    - [List configmaps:](#list-configmaps)
    - [Show the data from a ConfigMap:](#show-the-data-from-a-configmap)
  - [Creation](#creation)
    - [1. From literal values](#1-from-literal-values)
    - [2. From a file](#2-from-a-file)
    - [3. From a directory](#3-from-a-directory)
    - [4. From an environment variables file](#4-from-an-environment-variables-file)
  - [Consuming ConfigMaps](#consuming-configmaps)
    - [1. Individual environment variables](#1-individual-environment-variables)
    - [2. List of environment variables from a file](#2-list-of-environment-variables-from-a-file)
    - [3. ConfigMap as a volume](#3-configmap-as-a-volume)
    - [4. ConfigMap as one specific file](#4-configmap-as-one-specific-file)

## Describe the ConfigMap 

These examples show the output from the basic `get`/`describe` commands:

### List configmaps:

```bash
$  kubectl get configmaps 
NAME               DATA   AGE
kube-root-ca.crt   1      41m
db-config          3      35m
```

### Show the data from a ConfigMap:

Using the `get` command with the `-o yaml` flag:

```bash
$ kubectl get configmaps db-config -o yaml
apiVersion: v1
data:
  DB_HOST: db.example.com
  DB_NAME: SQL01
  DB_PORT: "3306"
kind: ConfigMap
(...)
```

Print detailed description with `describe`:

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

There are multiple ways to create ConfigMaps:
- [1. From literal values](#1-from-literal-values)
- [2. From a file](#2-from-a-file)
- [3. From a directory](#3-from-a-directory)
- [4. From an environment variables file](#4-from-an-environment-variables-file)

### 1. From literal values

To create ConfigMaps directly from the command line `kubectl create configmap <name> --from-literal=<key>=<value>`.

For example this new ConfigMap with name `webapp-config-map`:

```bash
$ kubectl create configmap webapp-config-map --from-literal=DB_NAME=SQL01
configmap/webapp-config-map created
```

### 2. From a file

To create ConfigMaps from a file `kubectl create configmap <name> --from-file=path/to/bar`.

Example:

```bash
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

This ConfigMap example is later used in [4. ConfigMap as one specific file](#4-configmap-as-one-specific-file) usage. 

### 3. From a directory

When the file points to a directory, each file directly in this directory is used as a key in the ConfigMap, where the content of the file is used as the value.

### 4. From an environment variables file

To create ConfigMaps from a environment variables file `kubectl create configmap <name> --from-env-file=path/to/bar`.

Example:

```bash
$ cat db-conf.env 
  DB_HOST=db.example.com
  DB_NAME=SQL01
  DB_PORT=3306

$ kubectl create configmap db-conf-cm --from-env-file=./db-conf.env 
configmap/db-conf-cm created
```

The keys that are considered invalid will be skipped. The pod will be allowed to start, but the invalid names will be recorded in the event log `InvalidVariableNames`, accessible from `kubectl get events`.

## Consuming ConfigMaps

There are multiple ways to consume ConfigMaps:
- [1. Individual environment variables](#1-individual-environment-variables)
- [2. List of environment variables from a file](#2-list-of-environment-variables-from-a-file)
- [3. ConfigMap as a volume](#3-configmap-as-a-volume)
- [4. ConfigMap as one specific file](#4-configmap-as-one-specific-file)

### 1. Individual environment variables

The `env` property specifies a **list of environment variables** to set in the container.

```yaml
apiVersion: v1
kind: Pod
metadata:
(...)
spec:
  containers:
  - image: nginx
    name: nginx
    env:
      - name: SPECIAL_KEY # name of the env variable inside the container
        valueFrom:
          configMapKeyRef:
            name: db-conf-cm # ConfigMap id
            key: special.key # Key inside the ConfigMap
```

### 2. List of environment variables from a file

The `envFrom` property specifies a **list of sources** to populate environment variables in the container.

```yaml
apiVersion: v1
kind: Pod
metadata:
(...)
spec:
  containers:
  - image: nginx
    name: nginx
    envFrom:
    - configMapRef:
      name: db-conf-cm 
```

### 3. ConfigMap as a volume

The `volumes` property specifies a **list of volumes** that can be mounted by containers in a pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
(...)
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - name: db-conf-volume # Same id used in volumes of the pod
      mountPath: /etc/config # Destination folder
  volumes:
  - name: db-conf-volume # Same id used in volumeMounts of the container
    configMap:
      name: db-conf-cm # ConfigMap id
```

Notes. 
- The `mountPath` must be an absolute path. 
- The destination folder must exist in the container image. 
- All files from the destination folder are removed and replaced by the ConfigMap.

### 4. ConfigMap as one specific file

The `volumeMounts.subPath` property specifies a sub-path inside the referenced volume instead of its root.

```yaml
...
spec:
  containers:
  - image: nginx
    name: nginx
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

```bash
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

