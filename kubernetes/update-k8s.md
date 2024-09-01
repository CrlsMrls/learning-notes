# Update Kubernetes

This document explains how to upgrade a Kubernetes cluster to a newer version using `kubeadm`. This concrete example upgrades from Kubernetes **v1.29** to **v1.30**. 

The output and steps are run in a **Debian-based** system. For other systems, refer to the [oficial Kubernetes documentation](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/), which provides instructions for different operating systems and different Kubernetes versions.

- [Update Kubernetes](#update-kubernetes)
  - [Remove applications from the controlplane node](#remove-applications-from-the-controlplane-node)
  - [Upgrade kubeadm package in the controlplane node](#upgrade-kubeadm-package-in-the-controlplane-node)
  - [Verify the upgrade plan](#verify-the-upgrade-plan)
  - [Apply the upgrade plan](#apply-the-upgrade-plan)
  - [Upgrade and restart Kubelet](#upgrade-and-restart-kubelet)
  - [Set the controlplane node as schedulable](#set-the-controlplane-node-as-schedulable)
  - [Update the remaining nodes](#update-the-remaining-nodes)
  - [Summary](#summary)


## Remove applications from the controlplane node
 
On the controlplane node:

```bash
kubectl drain controlplane --ignore-daemonsets 
```

Draining the node will evict all the pods running on the node. The `--ignore-daemonsets` flag will ignore the DaemonSet-managed pods.

Check the node is unscheduled:

```bash
kubectl get nodes
NAME           STATUS                     ROLES           AGE   VERSION
controlplane   Ready,SchedulingDisabled   control-plane   76m   v1.29.0
node01         Ready                      <none>          75m   v1.29.0

kubectl describe node controlplane | grep -i schedulable
Taints:             node.kubernetes.io/unschedulable:NoSchedule
Unschedulable:      true
  Normal  NodeNotSchedulable  16m    kubelet          Node controlplane status is now: NodeNotSchedulable
```

## Upgrade kubeadm package in the controlplane node

Update the Kubernetes apt repository from `/etc/apt/sources.list.d/kubernetes.list`:

```yaml
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /
```

Update the version in the URL to the next available minor release, i.e v1.30. After making changes, update the apt repository:

```bash
sudo apt update
```

Check the available versions of the kubeadm package:

```bash

sudo apt-cache madison kubeadm
  kubeadm | 1.30.4-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
  kubeadm | 1.30.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
  kubeadm | 1.30.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
  kubeadm | 1.30.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
  kubeadm | 1.30.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.30/deb  Packages
```

Based on the version information displayed by apt-cache madison, it indicates that for Kubernetes version 1.30.4, the available package version is `1.30.4-1.1`. Upgrade kubeadm to the desired version:

```bash
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.4-1.1' && \
sudo apt-mark hold kubeadm
```

Verify the version of kubeadm:

```bash
kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"30", GitVersion:"v1.30.4", GitCommit:"a51b3b711150f57ffc1f526a640ec058514ed596", GitTreeState:"clean", BuildDate:"2024-08-14T19:02:46Z", GoVersion:"go1.22.5", Compiler:"gc", Platform:"linux/amd64"}
```

## Verify the upgrade plan

```bash
sudo kubeadm upgrade plan v1.30.0
Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   NODE           CURRENT   TARGET
kubelet     controlplane   v1.29.0   v1.30.4
kubelet     node01         v1.29.0   v1.30.4

Upgrade to the latest stable version:

COMPONENT                 NODE           CURRENT    TARGET
kube-apiserver            controlplane   v1.29.0    v1.30.4
kube-controller-manager   controlplane   v1.29.0    v1.30.4
kube-scheduler            controlplane   v1.29.0    v1.30.4
kube-proxy                               1.29.0     v1.30.4
CoreDNS                                  v1.10.1    v1.11.1
etcd                      controlplane   3.5.10-0   3.5.12-0

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.30.4

_____________________________________________________________________
```


## Apply the upgrade plan 

Run the `apply` command to upgrade the Kubernetes cluster. Note that the above steps can take a few minutes to complete.

```bash
sudo kubeadm upgrade apply v1.30.4
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.30.4"
...
[upgrade/successful] SUCCESS! Your cluster was upgraded to "v1.30.4". Enjoy!

[upgrade/kubelet] Now that your control plane is upgraded, please proceed with upgrading your kubelets if you haven't already done so.
```

Once this is over, you can verify the upgrade: 

```bash
sudo kubeadm upgrade plan
[preflight] Running pre-flight checks.
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: 1.30.4
[upgrade/versions] kubeadm version: v1.30.4
I0901 09:25:27.543951   40720 version.go:256] remote version is much newer: v1.31.0; falling back to: stable-1.30
[upgrade/versions] Target version: v1.30.4
[upgrade/versions] Latest version in the v1.30 series: v1.30.4
```

## Upgrade and restart Kubelet

Now, upgrade the `kubelet` version. 

```bash
sudo apt-get install kubelet=1.30.4-1.1
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 26 not upgraded.
...
Unpacking kubelet (1.30.4-1.1) over (1.29.0-1.1) ...
dpkg: warning: unable to delete old directory '/etc/sysconfig': Directory not empty
Setting up kubelet (1.30.4-1.1) ...
```

Restart the kubelet service:

```bash
sudo systemctl daemon-reload

systemctl restart kubelet
```

## Set the controlplane node as schedulable

Verify the node status and its version:

```bash
kubectl get nodes
NAME           STATUS                     ROLES           AGE   VERSION
controlplane   Ready,SchedulingDisabled   control-plane   95m   v1.30.4
node01         Ready                      <none>          94m   v1.29.0
```

Mark the `controlplane` node as schedulable.

```bash
kubectl uncordon controlplane
node/controlplane uncordoned
```

Verify the node status:

```bash
kubectl get nodes
controlplane   Ready    control-plane   100m   v1.30.4
node01         Ready    <none>          99m    v1.29.0
```


## Update the remaining nodes

In the same way, update the remaining nodes. In the example, the `node01` node is updated.

First, drain the node to evict all the pods running there:

```bash
kubectl drain node01 --ignore-daemonsets
node/node01 cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-hfhnf, kube-system/kube-proxy-xhnng
evicting pod kube-system/coredns-7db6d8ff4d-xmcsl
evicting pod default/blue-667bf6b9f9-77qdp
evicting pod default/blue-667bf6b9f9-44jtq
evicting pod kube-system/coredns-7db6d8ff4d-j8ddj
pod/blue-667bf6b9f9-77qdp evicted
pod/blue-667bf6b9f9-44jtq evicted
pod/coredns-7db6d8ff4d-xmcsl evicted
pod/coredns-7db6d8ff4d-j8ddj evicted
node/node01 drained
```

SSH into the `node01` node and update the Kubernetes version as before:

1. Update the Kubernetes apt repository from `/etc/apt/sources.list.d/kubernetes.list`
2. Update the version in the URL to the next available minor release, i.e `1.30`.
3. Update the apt repository with `sudo apt update`
4. Check the available versions of the `kubeadm` package with `sudo apt-cache madison kubeadm`
5. Upgrade `kubeadm` application to the desired version with `sudo apt-get install kubeadm='1.30.4-1.1'`
6. Upgrade the node with `kubeadm upgrade node`
7. Uncordon the node with `kubectl uncordon node01`
8. Verify the node status with `kubectl get nodes`

Steps 1-5 are similar to the controlplane node upgrade.

After upgrading the node, the output should look like this:

```bash
kubeadm upgrade node
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Skipping prepull. Not a control plane node.
[upgrade] Skipping phase. Not a control plane node.
[upgrade] Backing up kubelet config file to /etc/kubernetes/tmp/kubeadm-kubelet-config123229599/config.yaml
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.
```

Steps 6-8 are also similar to the controlplane node upgrade, these steps are executed where `kubectl` is available.


```bash
kubectl get nodes
NAME           STATUS                     ROLES           AGE    VERSION
controlplane   Ready                      control-plane   103m   v1.30.4
node01         Ready,SchedulingDisabled   <none>          102m   v1.30.4

kubectl uncordon node01 
node/node01 uncordoned

kubectl get nodes
NAME           STATUS   ROLES           AGE    VERSION
controlplane   Ready    control-plane   104m   v1.30.4
node01         Ready    <none>          103m   v1.30.4
```

## Summary

In this example, we upgraded a Kubernetes cluster from v1.29 to v1.30. We first removed the applications from the controlplane node, upgraded the controlplane node, and then updated the remaining nodes. The steps involved updating the Kubernetes `apt` repository, upgrading the `kubeadm` package, applying the upgrade, and restarting the `kubelet` service. Finally, we marked the controlplane node as schedulable and updated the remaining nodes. The process ensures that the Kubernetes cluster is running the latest stable version while applications are evicted and rescheduled to maintain normal operation during the upgrade process.
