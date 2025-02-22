# Seccomp in Kubernetes

Seccomp (Secure Computing Mode) is a Linux kernel security feature that restricts the system calls (syscalls) available to a container. It helps mitigate attacks by reducing the attack surface within the kernel.

In Kubernetes, seccomp can be used to apply syscall filtering to containers, limiting their capabilities and enforcing security policies.


## Enabling Seccomp in Kubernetes

Seccomp support is enabled by default in Kubernetes starting from version 1.19. However, it requires explicit configuration to be enforced on workloads.

### Checking if Seccomp is Enabled

You can check if seccomp is enabled on your Kubernetes nodes by running:

```bash
kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.kubeletVersion}'
```

Ensure that your Kubernetes version is at least 1.19.

## Seccomp Profiles

Seccomp profiles define the set of syscalls that a container can use. The most common profiles are:

- `runtime/default`: The default profile applied by the container runtime.
- `unconfined`: No restrictions on syscalls.
- `localhost/<profile>`: A custom profile stored on the node.

## Applying Seccomp Profiles in Kubernetes

### Pod-Level Seccomp Configuration (Recommended)

Since Kubernetes 1.22, seccomp profiles can be defined in the `securityContext` field of a Pod specification.

#### Example: Applying a Seccomp Profile to a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-demo
  annotations:
    seccomp.security.alpha.kubernetes.io/pod: "runtime/default"
spec:
  containers:
  - name: seccomp-container
    image: busybox
    command: ["sleep", "3600"]
```

This example applies the `runtime/default` profile to the entire pod.

### Container-Level Seccomp Configuration

From Kubernetes 1.19 onwards, you can specify seccomp at the container level in the `securityContext`.

#### Example: Container-Level Seccomp Profile

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-container-demo
spec:
  containers:
  - name: app
    image: busybox
    securityContext:
      seccompProfile:
        type: RuntimeDefault
    command: ["sleep", "3600"]
```

## Custom Seccomp Profiles

To define a custom seccomp profile, create a JSON file on the node (e.g., `/var/lib/kubelet/seccomp/custom.json`) with the following content:

```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "architectures": ["SCMP_ARCH_X86_64"],
  "syscalls": [
    {
      "names": ["read", "write", "exit", "futex"],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}
```

Then reference the profile in your Pod spec:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: custom-seccomp-pod
spec:
  containers:
  - name: app
    image: busybox
    securityContext:
      seccompProfile:
        type: Localhost
        localhostProfile: "seccomp/custom.json"
    command: ["sleep", "3600"]
```

## Troubleshooting Seccomp Issues

- **Check Events:** Run `kubectl describe pod <pod-name>` to check if seccomp-related errors are causing pod failures.
- **Use Logs:** If a container fails due to syscall restrictions, check logs using `kubectl logs <pod-name>`.
- **Adjust Profiles:** If an application needs additional syscalls, modify the seccomp profile accordingly.

## Summary

- Seccomp restricts syscalls to enhance container security.
- Kubernetes supports seccomp via `securityContext` at pod and container levels.
- Use predefined profiles (`runtime/default`, `unconfined`) or create custom profiles.
- Apply seccomp policies carefully to balance security and functionality.

## References

- [Kubernetes Seccomp Documentation](https://kubernetes.io/docs/tutorials/security/seccomp/)
- [Linux Seccomp](https://man7.org/linux/man-pages/man2/seccomp.2.html)

