# Pod security Admissions and Pod Security Standards

Pod security standards are a set of best practices that can be enforced by the Kubernetes API server. These standards are enforced by the `PodSecurity` admission controller. 

This admission controller could be enabled by default in your cluster. If it is not enabled, you can enable it by passing the `--enable-admission-plugins` flag to the API server.

The following command verifies that the `PodSecurity` admission controller is enabled by default:

```bash
kubectl exec -n kube-system kube-apiserver-controlplane \
-- kube-apiserver -h | grep enable-admission
 --enable-admission-plugins strings   admission plugins that should be enabled in addition to default enabled ones (NamespaceLifecycle, ... PodSecurity, Priority, ...
```

## Default Pod Security Standards

Currently, there are three pod security standards available.

- **Privileged**: The Privileged policy is purposely-open, and entirely unrestricted.
- **Baseline**: Pods have some restrictions.
- **Restricted**: This is the most restrictive pod security standard. 

Check the [official documentation](https://kubernetes.io/docs/concepts/security/pod-security-standards/) for more details.

## Providing new admission controllers

In a Kubernetes cluster, the cluster administrator can specify the configuration file path for the admission configuration resource in the API server using the `--admission-control-config-file` flag.

This flag allows administrators to provide a file path that contains the configuration for admission controllers.



## Applying Pod Security Standards to a Namespace

### Warning Pod Security Standards

Apply a the label `pod-security.kubernetes.io/warn=` to a namespace will only warn about the pod security standard.

For example, warning the `baseline` on namespace `alpha`:

```bash
kubectl label ns alpha \
  pod-security.kubernetes.io/warn=baseline
```

If a pod is created in the `alpha` namespace that violates the `baseline` pod security standard, a warning is generated.

For example, creating a pod with a privileged security context:

```yaml
securityContext:
  privileged: true
```

The following warning is generated:

```bash
Warning: would violate PodSecurity "baseline:latest": privileged (container "baseline-pod" must not set securityContext.privileged=true)
pod/baseline-pod created
```

But the pod is still created.

#### Enforcing Pod Security Standards

To enforce the pod security standard, apply the label `pod-security.kubernetes.io/enforce=` to the namespace.

For example, enforcing the `restricted` pod security standard on the `alpha` namespace:

```bash
kubectl label ns alpha \
  pod-security.kubernetes.io/enforce=restricted
```

If a pod is created in the `alpha` namespace that violates the `restricted` pod security standard, the pod is not created.

```bash
$ kubectl run nginx --image nginx -n alpha
Error from server (Forbidden): pods "nginx" is forbidden: violates PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
```

#### Multiple Pod Security Standards

Multiple pod security standards can be applied to a namespace by separating them with a comma. 

For example, enforcing the `baseline` and warning the `restricted` pod security standards on the `beta` namespace:

```bash
kubectl label ns beta \
pod-security.kubernetes.io/enforce=baseline \
pod-security.kubernetes.io/warn=restricted
```

## Admission Configuration

Kubernetes provides a built-in admission controller to enforce the Pod Security Standards.  

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: AdmissionConfiguration
plugins:
  - name: PodSecurity
    configuration:
      apiVersion: pod-security.admission.config.k8s.io/v1
      kind: PodSecurityConfiguration
      defaults:
        enforce: baseline
        enforce-version: latest
        audit: restricted
        audit-version: latest
        warn: restricted
        warn-version: latest
      exemptions:
        usernames: [] 
        runtimeClassNames: [] 
        namespaces: [my-namespace] 
```

The previous object specifies to enforce the `baseline` pod security standard and audit/warn the `restricted` pod security standard.

The `exemptions` field specifies the namespaces that are exempt from the pod security standards, in this case, the `my-namespace` namespace.

