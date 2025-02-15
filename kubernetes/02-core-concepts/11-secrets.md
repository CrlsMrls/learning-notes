#  Secrets

Secrets objects allow to store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys. The secrets are stored in a `tmpfs` volume, which is mounted to the container. The secrets are stored in memory and never written to disk. The secrets are stored in base64-encoded format, which is not encrypted (as of today).

Aliases: `secret` or `secrets`



## Consuming Secrets from literal values

### Creation of the Secret object

The `kubectl create secret` command can be used to create a Secret object from literal values. The `--from-literal` flag is used to specify the key-value pairs.

```bash
$ kubectl create secret generic db-secret \
  --from-literal=DB_Host=sql01 \
  --from-literal=DB_User=root \
  --from-literal=DB_Password=password123
secret/db-secret created

$ kubectl describe secrets db-secret 
Name:         db-secret
Namespace:    secrets
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
DB_User:      4 bytes
DB_Host:      5 bytes
DB_Password:  11 bytes
```

### Consuming the secret literals

There are mainly two ways to consume the secret literals:
1. All key-value pairs
2. Individual key-value pairs

#### 1. All key-value pairs in a Secret as container environment variables

Use the `.spec.envFrom` field and the `secretRef` field to specify the Secret object:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envars-test
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 50"]
    envFrom:
    - secretRef:
        name: db-secret
```

After creating the pod, the environment variables are available in the container:

```bash
$ kubectl exec -it envars-test -- env | grep DB_
DB_Host=sql01
DB_Password=password123
DB_User=root
```

#### 2. Individual key-value pairs in a Secret as container environment variables

Use the `.spec.env` field and the `secretKeyRef` field to specify individual keys from the Secret object:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envars-test
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 50"]
    env:
    - name: DB_HOST
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: DB_Host
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: DB_User
```

After creating the pod, the environment variables are available in the container:

```bash
$ kubectl exec -it envars-test -- env | grep DB_
DB_HOST=sql01
DB_USER=root
```

## Consuming Secrets from files

### Creation of the Secret object

The `--from-file` flag is used to specify the key-value pairs. If not specified, the key is the filename and the value is the content of the file. 

```bash
$ cat ca.crt
-----BEGIN CERTIFICATE-----
MIIDXTCCAkWgAwIBAgIJAK7Z3Z3Z3Z3ZMA0GCSqGSIb3DQEBCwUAMEUxCzAJBgNV
BAYTAkFVMRMwEQYDVQQIDApTb21lLVN0YXRlMQ8wDQYDVQQKDAZNeUZpbGUxETAP
...
-----END CERTIFICATE-----

$ kubectl create secret generic certificate-secret \
  --from-file=ca=./ca.crt 
secret/certificate-secret created

$ kubectl describe secrets certificate-secret 
Name:         certificate-secret
Namespace:    secrets
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
ca:  188 bytes
```

### Consuming the secret files

There are mainly two ways to consume the secret files:
1. As a volume
2. As a file in a volume

#### 1. As a volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: certificate-test
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 50"]
    volumeMounts:
    - name: certificate-volume
      mountPath: /etc/ssl/cert
      readOnly: true
  volumes:
  - name: certificate-volume
    secret:
      secretName: certificate-secret
```

After creating the pod, the files are available in the container:

```bash
$ kubectl exec -it certificate-test -- ls -l /etc/ssl/cert
total 0
lrwxrwxrwx    1 root     root             9 Sep 22 13:45 ca -> ..data/ca

$ kubectl exec -it certificate-test -- cat /etc/ssl/cert/ca
-----BEGIN CERTIFICATE-----
MIIDXTCCAkWgAwIBAgIJAK7Z3Z3Z3Z3ZMA0GCSqGSIb3DQEBCwUAMEUxCzAJBgNV
BAYTAkFVMRMwEQYDVQQIDApTb21lLVN0YXRlMQ8wDQYDVQQKDAZNeUZpbGUxETAP
...
-----END CERTIFICATE-----
```

### 2. As a file in a volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: certificate-test
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "sleep 50"]
    volumeMounts:
    - name: certificate-volume
      mountPath: /etc/ssl/cert/ca.crt # filename in the volume
      subPath: ca # must match the key in the Secret object
      readOnly: true
  volumes:
  - name: certificate-volume
    secret:
      secretName: certificate-secret
```

**Note** `SubPaths` are not automatically updated when the Secret is modified. If the Secret is updated, the pod must be deleted and recreated.

After creating the pod, the files are available in the container:

```bash
$ kubectl exec -it certificate-test -- ls -l /etc/ssl/cert
total 4
-rw-r--r--    1 root     root           1119 Sep  9 14:58 ca.crt
```

## References

- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)