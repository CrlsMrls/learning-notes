# Admission controller

Kubernetes admission controller is a piece of code that intercepts requests to the Kubernetes API server prior to persistence of the object, but after the request is authenticated and authorized. Admission controllers are the primary way to extend the behavior of the Kubernetes API server to enforce security, compliance, and business rules.

Admission is divided into two phases: In the first phase, only mutating admission plugins run. In the second phase, only validating admission plugins run.

## Enable or disable admission controllers

Admission controllers are enabled by passing the `--enable-admission-plugins` flag to the API server. The flag takes a comma-separated list of admission controllers to enable. The order of the controllers in the list is significant because the controllers are executed in the order they are specified.

Default adminssion controlles can be dissabled by passing the `--disable-admission-plugins` flag to the API server.

## List enabled admission controllers

To validate the cluster has admission controllers enabled, you can use the `kubectl api-versions` command.


```bash
$ kubectl api-versions | grep admission

admissionregistration.k8s.io/v1
```

## List of default admission controllers

Each Kubernetes cluster has a set of default admission controllers that are enabled by default. Execute the following command to list the default admission controllers enabled in your cluster:

```bash
$ kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'enable-admission-plugins'

  --enable-admission-plugins strings  admission plugins that should be enabled in addition to default enabled ones (NamespaceLifecycle, LimitRanger, ServiceAccount, ...). Comma-delimited list of admission plugins: AlwaysAdmit, AlwaysDeny, AlwaysPullImages, CertificateApproval, CertificateSigning, ... 
```

## Creating a custom admission webhook congifuration

To create a custom admission webhook configuration, you need to create a `ValidatingWebhookConfiguration` or `MutatingWebhookConfiguration` object. The configuration object specifies the URL of the webhook server, the resources and operations the webhook should be invoked for, and the client configuration for the webhook.

The following is an example of a `ValidatingWebhookConfiguration` object:

```yaml
piVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: demo-webhook
webhooks:
  - name: webhook-server.webhook-demo.svc
    clientConfig:
      service:
        name: webhook-server
        namespace: webhook-demo
        path: "/mutate"
      caBundle: zGVtby1jYS1iYXNlLXRlc3Q=
    rules:
      - operations: [ "CREATE" ]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
    admissionReviewVersions: ["v1"]
    sideEffects: None
```

This configuration specifies that the webhook server should be invoked for the `CREATE` operation on pods. The webhook server is located at `webhook-server.webhook-demo.svc` in the `webhook-demo` namespace. 

Possible use cases for this example of webhook 
- all pods must run as non-root user, applying a specific security context to the pod.
- all pods must have a specific annotation or label, rejecting the pod creation if the annotation or label is not present.
- etc.









