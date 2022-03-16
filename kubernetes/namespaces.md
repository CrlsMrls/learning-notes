# namespaces

`namespaces` isolates groups of resources within a cluster.

## Common namespaces

### default

By default everything is grouped under the `default` namespace.

### kube-system

`kube-system` namespace groups all the essential k8s services:

- `coredns` (replaces kube-dns as of 1.11)
- `kube-scheduler` and `kube-controller-manager` (the control plane pods)
- `kube-apiserver`
- `etcd`
- etc.

### kube-public

`kube-public` is created by the installer and eventually used for security bootstraping
