# Whitelist Allowed Registries

This document describes the process for adding a new registry to the list of allowed registries in the Kubernetes project.

## ImagePolicyWebhook

The `ImagePolicyWebhook` is an admission controller that validates the image registry of a pod. 

Use cases:
- Whitelist allowed registries.
- Blacklist disallowed registries.
- Enforce tagging conventions, e.g. `latest` tag is not allowed.

## Basic Webhook server

The following is an example of a basic webhook server that validates the image registry of a pod and rejects all the pods that are using images with the `latest` tag [source](https://github.com/kainlite/kube-image-bouncer):

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: image-bouncer-webhook
  name: image-bouncer-webhook
spec:
  type: NodePort
  ports:
    - name: https
      port: 443
      targetPort: 1323
      protocol: "TCP"
      nodePort: 30080
  selector:
    app: image-bouncer-webhook
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: image-bouncer-webhook
spec:
  selector:
    matchLabels:
      app: image-bouncer-webhook
  template:
    metadata:
      labels:
        app: image-bouncer-webhook
    spec:
      containers:
        - name: image-bouncer-webhook
          imagePullPolicy: Always
          image: "kainlite/kube-image-bouncer:latest"
          args:
            - "--cert=/etc/admission-controller/tls/tls.crt"
            - "--key=/etc/admission-controller/tls/tls.key"
            - "--debug"
            - "--registry-whitelist=docker.io,registry.k8s.io"
          volumeMounts:
            - name: tls
              mountPath: /etc/admission-controller/tls
      volumes:
        - name: tls
          secret:
            secretName: tls-image-bouncer-webhook
```

The webhook server is deployed as a `Deployment` and exposed as a `Service` with a NodePort type. The webhook server validates the image registry of a pod against the whitelist of allowed registries, in this case, `docker.io` and `registry.k8s.io`.

## Admission Configuration

To enable the ImagePolicyWebhook, you need to create a `ValidatingWebhookConfiguration` object. The configuration object specifies the URL of the webhook server, the resources and operations the webhook should be invoked for, and the client configuration for the webhook.

The following is an example of a `ValidatingWebhookConfiguration` object:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: ImagePolicyWebhook
  configuration:
    imagePolicy:
      kubeConfigFile: /etc/kubernetes/pki/admission_kube_config.yaml
      allowTTL: 50
      denyTTL: 50
      retryBackoff: 500
      defaultAllow: false
```

In this case, stored in `/etc/kubernetes/pki/admission_configuration.yaml` file.


Following the example above, the `admission_kube_config.yaml` file could look like this:

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/server.crt
    server: https://image-bouncer-webhook:<NODE_PORT>/image_policy
  name: bouncer_webhook
contexts:
- context:
    cluster: bouncer_webhook
    user: api-server
  name: bouncer_validator
current-context: bouncer_validator
preferences: {}
users:
- name: api-server
  user:
    client-certificate: /etc/kubernetes/pki/apiserver.crt
    client-key:  /etc/kubernetes/pki/apiserver.key
```


## Enabling the ImagePolicyWebhook

Enable the `ImagePolicyWebhook` admission controller as final step so that our image policy validation can take place in API server.

Add the `--admission-control-config-file` as well for this controller.


First check if the `ImagePolicyWebhook` is enabled by default:

```bash
$ kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'ImagePolicyWebhook'
   --enable-admission-plugins strings     admission plugins that should be enabled in addition to default enabled ones (NamespaceLifecycle, ...). Comma-delimited list of admission plugins: AlwaysAdmit, ... ImagePolicyWebhook, ...
```

Which means that the `ImagePolicyWebhook` is not enabled by default.

Aftwerards, validate the config file parameter:

```bash
$ kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'admission-control-config-file'
  --admission-control-config-file string   File with admission control configuration.
```

Finally, enable the `ImagePolicyWebhook` by adding the `--admission-control-config-file` flag to the kube-apiserver manifest file:

```bash
$ vim /etc/kubernetes/manifests/kube-apiserver.yaml

...

spec:
  containers:
  - command:
    - kube-apiserver
    - --admission-control-config-file=/etc/kubernetes/pki/admission_configuration.yaml
    - --enable-admission-plugins=NodeRestriction,ImagePolicyWebhook
```

Trying to create a pod with an image from a disallowed registry will result in the following error:

```bash
  Warning  FailedCreate  27s (x5 over 65s)  replicaset-controller  (combined from similar events): Error creating: pods "nginx-latest-2jjg8" is forbidden: image policy webhook backend denied one or more images: Images using latest tag are not allowed
```