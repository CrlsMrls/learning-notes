# Kubelet

Kubelet is the primary "node agent" that runs on each node. It can register the node with the API server using one of: the hostname; a flag to override the hostname; or specific logic for a cloud provider.

## Configuration

The Kubelet configuration defines how the Kubelet behaves. It controls resource limits, security settings, logging, networking, and other operational parameters that affect how Pods are scheduled and run.

The Kubelet configuration is located at the configmap `kubelet-config` in the `kube-system` namespace. The configuration file stored in the `/var/lib/kubelet/config.yaml` file is a local cache of the configuration.

Modifying the configuration file directly is not recommended. Instead, use the `kubectl` command to update the configuration.


```bash
$ kubectl edit cm kubelet-config -n kube-system
$ systemctl restart kubelet
```

Verify the configuration changes by checking the Kubelet logs:

```bash
$ journalctl -u kubelet
```

## Updating the Kubelet Configuration in a kubeadm Cluster

In a `kubeadm` cluster, the Kubelet configuration is managed by the `kubeadm` tool. To update the Kubelet configuration, use the `kubeadm` command:

```bash
$ kubeadm upgrade node phase kubelet-config 
$ systemctl restart kubelet
```

Running the previous command will update the Kubelet configuration to match the control plane version. 

Repeat the process on all nodes in the cluster.











