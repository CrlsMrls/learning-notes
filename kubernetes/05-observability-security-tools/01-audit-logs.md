# Kubernetes audit logs

Kubernetes has an audit feature that logs all requests made to the Kubernetes API server. The audit logs can be used for security, compliance, and investigating security incidents.

Because all requests must go through the API server, the audit logs provide a comprehensive view of all activities in the cluster. They can be crucial for tracking security incidents, allowing security teams to analyze logs to provide evidence of unauthorized actions.


## Audit Policy

The audit policy defines the rules for logging requests to the Kubernetes API server. The policy is defined in a YAML file and can be passed to the API server using the `--audit-policy-file` flag.

The audit policy file has the following structure:

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
```

The previous example would log all API requests at the `Metadata` level. 


## Audit Level

The `level` field specifies the level of logging for the request. The following levels are available:
- `None`: Do not log the request.
- `Metadata`: Log the request metadata.
- `Request`: Log the request metadata and request body.
- `RequestResponse`: Log the request metadata, request body, and response body.

## Stages

The audit policy can be applied to different stages of the request. The following stages are available:
- `RequestReceived`: Log when the request is received by the API server.
- `ResponseStarted`: Once the response starts.
- `ResponseComplete`: After the response is complete.
- `Panic`: When the API server panics due to an internal error.

These stages can be combined to log different stages of the request. For example, only log the request metadata when for non-important requests, and log request metadata, body, and response for critical operations..

## Examples of an Audit Policy

The following example logs all requests to `delete` and `create` `secrets` in the `prod` namespace at the `Metadata` level:

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  namespaces: ["prod"]
  verbs: ["delete", "create"]
  resources:
  - group: ""
    resources: ["secrets"]
```

The following example logs all requests (after they are received) to `secrets` and `configmaps` resources in the `prod` and `uat` namespaces at the `Metadata` level:

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:             # Omit RequestReceived
  - RequestReceived
rules:
  - level: Metadata     # New rule at Metadata level
    resources:          # for secrets and configmaps
    - group: ""
      resources:
      - secrets
      - configmaps
    namespaces:         # in all two namespaces
    - prod
    - uat
```

## Enabling Audit Logs

Audit logs can be stored in a file (`--audit-log-path`), sent to a webhook (`--audit-webhook-config-file`), or streamed to external log collectors like Elasticsearch, Loki, or Fluentd. 

To enable audit logs storing in a file, the `--audit-policy-file` flag must be passed to the API server. The flag takes the path to the audit policy file as an argument.

```yaml
...
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --profiling=false
    - --audit-policy-file=/etc/kubernetes/prod-audit.yaml
    - --audit-log-path=/var/log/kubernetes/audit/prod-audit.log
    - --audit-log-maxage=30
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
...

volumeMounts:
- mountPath: /etc/kubernetes/prod-audit.yaml
  name: audit-policy
  readOnly: true
- mountPath: /var/log/kubernetes/audit/
  name: audit-log
  readOnly: false
...
volumes:
  volumes:
  - name: audit-policy
    hostPath:
      path: /etc/kubernetes/prod-audit.yaml
      type: File
  - name: audit-log
    hostPath:
      path: /var/log/kubernetes/audit/
      type: DirectoryOrCreate

```

In the previous example, the audit policy file is mounted as a volume in the API server container. The audit logs are stored in the `/var/log/kubernetes/audit/` directory.

Note: Higher logging levels (especially `RequestResponse`) can increase disk usage and impact API server performance. It's recommended to log only sensitive actions at higher levels and use the flags:
- `--audit-log-maxage`: number of days to retain audit log files, 
- `--audit-log-maxbackup`: number of audit log files to retain, and
- `--audit-log-maxsize`: size in megabytes of the audit log file before it gets rotated


## Viewing Audit Logs

Following the previous example, the audit logs are stored in the `/var/log/kubernetes/audit/` directory from the host machine where the API server is running.

```bash
$ sudo cat /var/log/kubernetes/audit/prod-audit.log
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"16c4f91b-c657-4ea8-acae-69873850d819","stage":"RequestReceived","requestURI":"/api/v1/namespaces/prod/secrets/some-secret","verb":"delete","user":{"username":"kubernetes-admin","groups":["kubeadm:cluster-admins","system:authenticated"]},"sourceIPs":["192.168.63.171"],"userAgent":"kubectl/v1.31.0 (linux/amd64) kubernetes/9edcffc","objectRef":{"resource":"secrets","namespace":"prod","name":"some-secret","apiVersion":"v1"},"requestReceivedTimestamp":"2025-02-21T11:23:14.940487Z","stageTimestamp":"2025-02-21T11:23:14.940487Z"}
{"kind":"Event","apiVersion":"audit.k8s.io/v1","level":"Metadata","auditID":"16c4f91b-c657-4ea8-acae-69873850d819","stage":"ResponseComplete","requestURI":"/api/v1/namespaces/prod/secrets/some-secret","verb":"delete","user":{"username":"kubernetes-admin","groups":["kubeadm:cluster-admins","system:authenticated"]},"sourceIPs":["192.168.63.171"],"userAgent":"kubectl/v1.31.0 (linux/amd64) kubernetes/9edcffc","objectRef":{"resource":"secrets","namespace":"prod","name":"some-secret","apiVersion":"v1"},"responseStatus":{"metadata":{},"status":"Success","details":{"name":"some-secret","kind":"secrets","uid":"b2368870-35a7-43e2-b2b1-c69eec9103a2"},"code":200},"requestReceivedTimestamp":"2025-02-21T11:23:14.940487Z","stageTimestamp":"2025-02-21T11:23:14.943654Z","annotations":{"authorization.k8s.io/decision":"allow","authorization.k8s.io/reason":"RBAC: allowed by ClusterRoleBinding \"kubeadm:cluster-admins\" of ClusterRole \"cluster-admin\" to Group \"kubeadm:cluster-admins\""}}
```

In the previous example, the audit logs show a request to delete a secret in the `prod` namespace. The request was made by the `kubernetes-admin` user and was successful.


More info about the audit policy can be found in the [official documentation](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/).

## 📚 References

- [Kubernetes Audit Policy Reference](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)