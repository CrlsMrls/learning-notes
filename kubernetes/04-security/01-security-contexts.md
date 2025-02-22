# Security Contexts  

Security Contexts allow containers and pods to run with specific security configurations, defining privileges, access levels, and system constraints to enhance security.  

## 📌 Pod Security Contexts  

Security contexts can be configured at the **pod level** or the **container level** in a pod manifest.  

### 🛡️ Common Security Context Fields  

| Field | Description |
|-------|------------|
| `privileged` | Grants full access to the host's resources when set to `true`. **Use with extreme caution.** |
| `allowPrivilegeEscalation` | Prevents a process from gaining more privileges than its parent. **Defaults to `true` unless explicitly set to `false`**. |
| `runAsUser` | Sets the user ID under which all processes in the container will run. Defaults to the container runtime’s user if not set. |
| `runAsGroup` | Sets the primary group ID for the container's processes. Defaults to `runAsUser` if not specified. |
| `fsGroup` | Defines the group ID that owns all volumes mounted to the pod. |
| `supplementalGroups` | Specifies additional group IDs for the first process in each container. |
| `seLinuxOptions` | Defines SELinux labels for the container. |
| `readOnlyRootFilesystem` | Enforces a read-only root filesystem for the container. |
| `capabilities` | Defines Linux capabilities to add or drop from the container. |
| Seccomp | Filter a process's system calls. |
| AppArmor | Restrict the capabilities of individual programs. |


## 🔧 Setting Security Contexts  

Security contexts can be defined **at the pod level** (applying defaults to all containers) or **at the container level** (overriding pod-level settings if needed).  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: constrained-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 2000
  containers:
  - name: sec-ctx
    image: busybox
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
      readOnlyRootFilesystem: true
```

If a container has its own securityContext, it will override the pod’s settings for that container only. In the previous example, the `sec-ctx` container runs as user 2000, while the other containers in the pod run as user 1000.


## 🏗️ Security Contexts for Kubernetes Control Plane
Unlike application pods, control plane components (kube-apiserver, kube-scheduler, etc.) run as static pods in the /etc/kubernetes/manifests/ directory. Their security settings must be configured in the YAML manifests before starting the cluster.

For example, to restrict kube-apiserver from running as root, modify its manifest:

```yaml
spec:
  containers:
  - name: kube-apiserver
    securityContext:
      runAsUser: 1000
      runAsGroup: 1000
      fsGroup: 2000
      allowPrivilegeEscalation: false
```

**💡 Note:** kubeadm does not apply security contexts to control plane components by default. This means they might run with elevated privileges unless explicitly configured.

## 🚀 Best Practices
- ✅ Enforce least privilege: Avoid `privileged: true` and limit capabilities with `capabilities.drop`.
- ✅ Run as non-root: Set `runAsUser` and avoid running as UID 0.
- ✅ Restrict privilege escalation: Set `allowPrivilegeEscalation: false` for all containers.
- ✅ Enable read-only filesystems: Use `readOnlyRootFilesystem: true` when possible.
- ✅ Use Pod Security Standards (PSS) and Admission Controllers: Restrict unsafe configurations using policies. 

## 📚 References

- [Kubernetes Security Context Documentation](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)