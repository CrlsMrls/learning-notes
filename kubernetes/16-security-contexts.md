# Security Contexts

Security Contexts allow to run containers with different privileges, limiting what processes running in containers can do, controlling access to the host OS, and providing a secure environment for these processes.

## Pod Security Contexts

Security contexts for pods can be configured in the pod manifest. 

These are the most common security contexts:
- `runAsUser`: Specifies that for any Containers in the Pod, all processes run with a specific user ID. If this is not specified, the `kubelet` will assign a user ID for each container.
- `runAsGroup`: Specifies the primary group ID of the container process. If unspecified, it defaults to the `runAsUser` value.
- `fsGroup`: Specifies the owner of any volume attached to the pod. If unspecified, it defaults to the `runAsUser` value.
- `supplementalGroups`: Specifies the list of groups applied to the first process run in each container, in addition to the `runAsGroup`. If unspecified, no groups are added to any container.
- `seLinuxOptions`: Specifies the SELinux context for the container.
- `allowPrivilegeEscalation`: Specifies whether a process can gain more privileges than its parent process. 
- `readOnlyRootFilesystem`: Sets the container's root filesystem is read-only.
- `linuxCapabilities`: Specifies a list of capabilities that are added or removed from the default set for a container.
- `privileged`: Specifies whether a container can access host devices without any restrictions. Always use this with caution.
- `capabilities`: Specifies a list of capabilities that are added or removed from the default set for a container.

## Setting Security Contexts

`securityContext` can be used at pod `spec` and at container level. The security context for a Pod applies to the Pod's Containers and also to the Pod's Volumes when applicable.

When used at the container, they apply only to the individual Container, and they override settings made at the Pod level when there is overlap.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: constrained-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: sec-ctx
    image: busybox
    securityContext:
      runAsUser: 2000
      allowPrivilegeEscalation: false
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```

## Control Plane Security Contexts

To configure security contexts for the control plane components, you need to edit the manifests before starting the cluster. For example, to run the `kube-apiserver` as a non-root user, you need to edit the `kube-apiserver.yaml` manifest and add the following lines:

```yaml
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
```

## Notes

- `kubeadm` does not configure any security context for the control plane components. This allows pods any possible elevated privileges by default. For example, the pod can run as root, access the host filesystem, modify the host network, etc.


****
## References

- [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)
- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)