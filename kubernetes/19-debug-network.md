# Debugging Kubernetes network issues

The common places to look for errors related to CNI plugins are the kubelet logs, CNI plugin logs, container runtime logs, and pod status.


## Troubleshooting steps

These are the steps to debug network issues in a Kubernetes cluster:
- Start with the pod status using `kubectl describe pods <pod-name>` to see if the pod is stuck in the `Pending` state because of a network issue.
- Continue with `kubelet` logs using `journalctl -u kubelet` to identify CNI-related errors.
- Alternatively, check the container runtime logs using `journalctl -u containerd` or `journalctl -u docker`.
- Finally, check logs for the specific CNI plugin produced by the pods in the `kube-system` namespace using `kubectl logs`.
- Optinally, some CNI plugins log directly to the node's file system, explore `/var/log/` directories on the node for plugin-specific log files.


## 1. Pod status

If the network layer is not correctly set up for the pod, the pod status may be stuck in the `Pending` or `ContainerCreating` state. However, networking issues can also occur after the pod has started running, leading to states like `CrashLoopBackOff` or even affecting seemingly Running pods that lack proper connectivity. These issues can be due to misconfigured network policies, IP conflicts, or CNI plugin errors.

```bash
$ kubectl describe pods app
...
 Status:        Pending
...
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedCreatePodSandBox  11m                 kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = failed to setup network for sandbox "d97...965c1": plugin type="weave-net" name="weave" failed (add): unable to allocate IP address: Post "http://127.0.0.1:6784/ip/d97...965c1": dial tcp 127.0.0.1:6784: connect: connection refused
```

In the example above, the pod status is in the `Pending` state because the `kubelet` failed to set up the network for the pod. The error message indicates that the `weave-net` plugin failed to allocate an IP address for the pod.
 

## 2. CNI plugin logs

The container runtime, guided by the `kubelet`'s instructions via the Container Runtime Interface (CRI), is responsible for setting up and tearing down pod networks. It does so by invoking the appropriate CNI plugin. While errors related to network setup might appear in `kubelet` logs, detailed debugging often requires checking the logs of the specific CNI plugin components (e.g., `calico-node`, `weave-net`).

Use the `journalctl` command to view the logs of the `kubelet` service.

```bash
$ journalctl -u kubelet
...
pod_workers.go:1301] "Error syncing pod, skipping" err="failed to \"KillPodSandbox\" for \"48f5de46-500c-4fbb-bf1c-604133698e83\" with KillPodSandboxError: \"rpc error: code = Unknown desc = failed to destroy network for sandbox \\\"d97...965c1\\\": plugin type=\\\"weave-net\\\" name=\\\"weave\\\" failed (delete): Delete \\\"http://127.0.0.1:6784/ip/d97...965c1\\\": dial tcp 127.0.0.1:6784: connect: connection refused\"" pod="default/app" podUID="48f5de46-500c-4fbb-bf1c-604133698e83"
```

In the example above, the `kubelet` service logged an error message because the `weave-net` plugin failed to destroy the network for the pod.

## 3. Container runtime logs

The container runtime will log an error message if it encounters issues invoking the CNI plugin or loading the network configuration. This often happens when the CNI configuration file is missing, malformed, or incompatible with the runtime.

```bash
$ journalctl -u containerd
...
level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"
```

In the example above, the `containerd` service logged an error message because the CNI configuration file was missing.

The `/etc/cni/net.d/` directory might have a different path depending on the configuration. Some systems might use `/opt/cni/net.d/` or a custom path defined during setup.

## 4. CNI pod logs

Some CNI plugins use Kubernetes pods/containers for their components, which may have logs accessible via `kubectl`.

1. Weave `kubectl logs -n kube-system -l name=weave-net` 
2. Calico `kubectl logs -n kube-system -l k8s-app=calico-node`
3. Flannel: `kubectl logs -n kube-system -l app=flannel`
4. Cilium: `kubectl logs -n kube-system -l k8s-app=cilium`

CNI plugins may have other components. For example, Calico might have logs in the `calico-typha` deployment (if deployed). On the other hand, Cilium has an operator with detailed logs.

Finally, not all CNI plugins log to Kubernetes pods. Some log directly to the filesystem (e.g., `/var/log/`) or use custom logging mechanisms. Refer to the specific CNI plugin documentation for more details.

Check the file [Linux network discovery](../Linux/networking-discovery.md) for more information on how to troubleshoot network issues on Linux systems.

## Common Causes of Network Issues in a Kubernetes Cluster

### 1. CNI Plugin Misconfigurations:

- Missing or incorrect files in `/etc/cni/net.d/`.
- Exhausted IP ranges due to improper IPAM settings. 
- Overlapping subnets causing IP conflicts.
- Version mismatches between the CNI plugin, Kubernetes, or the container runtime.

### 2. Pod-Level Problems:

- Pods stuck in the `Pending` state because of network setup failures.
- Misconfigured network policies blocking traffic between pods.
- DNS misconfigurations within the pod (e.g., an incorrect `/etc/resolv.conf`) can also cause issues.

### 3. Node-Level Issues:

- Routing table or firewall misconfigurations on nodes.
- MTU mismatches causing packet loss.
- Dynamic IP changes for nodes in cloud environments.
- Check kernel logs (`dmesg`) for network-related errors on the node.

### 4. Service-Related Issues:

- Incorrect Kubernetes Service configurations (e.g., bad selectors or ports).
- DNS resolution issues with CoreDNS.

### 5. External Connectivity Problems:

- Firewall or cloud security group restrictions blocking traffic.
- Proxy misconfigurations affecting outbound/inbound communication.

### 6. Cluster Component Failures:

- `Kube-proxy` misconfigurations disrupting service traffic.
- Crashes or misconfigurations in CNI plugin pods (e.g., `Calico`, `Weave`, `Flannel`).


