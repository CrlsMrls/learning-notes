# namespaces

`namespaces` isolates groups of resources within a cluster.

Aliases: `namespaces` and `ns`

- [namespaces](#namespaces)
  - [Namespace creation](#namespace-creation)
  - [Creating resources in a specific namespace](#creating-resources-in-a-specific-namespace)
  - [Common namespaces](#common-namespaces)
    - [default](#default)
    - [kube-system](#kube-system)
    - [kube-public](#kube-public)
  - [Get information on namespaces](#get-information-on-namespaces)

## Namespace creation

```bash
$ kubectl create namespace dev-ns --dry-run=client -o yaml > namespace-definition.yml

$ cat namespace-definition.yml
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: null
  name: dev-ns
spec: {}
status: {}

$ kubectl apply -f namespace-definition.yml
namespace/dev-ns created
```

## Creating resources in a specific namespace

Add `.metadata.namespace: <namespace-name>` in the description file.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: frontend
  namespace: dev
  labels:
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```

In the previous example, the pod will be created in the `dev` namespace.


## Common namespaces

### default

By default everything is grouped under the `default` namespace. It is created automatically when the cluster is first setup.

### kube-system

`kube-system` namespace groups all the essential k8s services:

- `coredns` (replaces kube-dns as of 1.11)
- `kube-scheduler` and `kube-controller-manager` (the control plane pods)
- `kube-apiserver`
- `etcd`
- etc.

### kube-public

`kube-public` is created by the installer and eventually used for security bootstraping

## Get information on namespaces

Getting all namespaces in a cluster:

```bash
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   46m57s
kube-system       Active   46m57s
kube-public       Active   46m57s
research          Active   46m57s
```

Add `--namespace=<namespace-name>` or `-n=<namespace-name>` for getting resources in a specific namespace.

```bash
$ kubectl get pods --namespace=research
NAME    READY   STATUS      RESTARTS      AGE
dna-1   0/1     Completed   4 (62s ago)   52m11s
dna-2   0/1     Completed   4 (59s ago)   52m11s
```

Add `--all-namespaces` or `-A` for listing resources in all namespaces.

```bash
$ kubectl get pods --all-namespaces
NAMESPACE       NAME                                     READY   STATUS             RESTARTS      AGE
kube-system     helm-install-traefik-crd--1-mlsch        0/1     Completed          0             28m
kube-system     helm-install-traefik--1-lddpr            0/1     Completed          1             28m
kube-system     traefik-74dd4975f9-6v546                 1/1     Running            0             27m
kube-system     local-path-provisioner-64ffb68fd-c4h84   1/1     Running            0             28m
kube-system     coredns-85cb69466-xb2vj                  1/1     Running            0             28m
kube-system     metrics-server-9cf544f65-6zcpn           1/1     Running            0             28m
...
```
