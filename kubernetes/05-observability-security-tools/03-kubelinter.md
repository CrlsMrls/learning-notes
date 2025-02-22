# Static Analysis with KubeLinter

KubeLinter is an open-source static analysis tool that checks Kubernetes YAML manifests, Helm charts, and Kustomize configurations for security misconfigurations.

- Identifies security risks in Kubernetes workloads before deployment.
- Helps enforce best practices for RBAC, securityContext, and network policies.
- Can be integrated into CI/CD pipelines to prevent misconfigurations.

## Installation

Download the latest release of KubeLinter from the GitHub repository:

```bash
curl -LO https://github.com/stackrox/kube-linter/releases/latest/download/kube-linter-linux.tar.gz
tar -xvf kube-linter-linux.tar.gz
chmod +x kube-linter
sudo mv kube-linter /usr/local/bin/
```

## Analyzing Kubernetes Manifests

Lint a Kubernetes manifest file using the following command:

```bash
kube-linter lint deployment.yaml
```

The output displays the issues found in the manifest file.

Example output:

```bash
$ kube-linter lint nginx.yml 
KubeLinter 0.7.1

/root/nginx.yml: (object: <no namespace>/nginx-deployment apps/v1, Kind=Deployment) object has 3 replicas but does not specify inter pod anti-affinity (check: no-anti-affinity, remediation: Specify anti-affinity in your pod ...)

/root/nginx.yml: (object: <no namespace>/nginx-deployment apps/v1, Kind=Deployment) container "nginx" does not have a read-only root file system (check: no-read-only-root-fs, remediation: Set readOnlyRootFilesystem to true in the container securityContext.)

/root/nginx.yml: (object: <no namespace>/nginx-deployment apps/v1, Kind=Deployment) container "nginx" is not set to runAsNonRoot (check: run-as-non-root, remediation: Set runAsUser to a non-zero number and runAsNonRoot to true in your pod or container securityContext. ...)

/root/nginx.yml: (object: <no namespace>/nginx-deployment apps/v1, Kind=Deployment) container "nginx" has cpu request 0 (check: unset-cpu-requirements, remediation: Set CPU requests for your container ...)

/root/nginx.yml: (object: <no namespace>/nginx-deployment apps/v1, Kind=Deployment) container "nginx" has memory limit 0 (check: unset-memory-requirements, remediation: Set memory limits for your container ...)

Error: found 5 lint errors
```

### Remediation

To fix the issues found by KubeLinter, update the manifest file with the recommended remediation steps.

For example, to fix the issue with the CPU request, update the manifest file with the following configuration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - nginx
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        securityContext:
          readOnlyRootFilesystem: true
          runAsUser: 1000
          runAsNonRoot: true
```

After updating the manifest file, run KubeLinter again to verify that the issues have been resolved.

```bash
kube-linter lint nginx.yml 
KubeLinter 0.7.1

No lint errors found!
```