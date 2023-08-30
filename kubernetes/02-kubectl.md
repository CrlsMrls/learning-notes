# kubectl

kubectl is the command line tool for communicating with a Kubernetes cluster, it connects to control pane's API.

- [kubectl](#kubectl)
  - [Configuration](#configuration)
  - [Common commands](#common-commands)
    - [kubectl run](#kubectl-run)
    - [kubectl apply](#kubectl-apply)
    - [kubectl delete](#kubectl-delete)
  - [Get information](#get-information)
    - [kubectl describe ](#kubectl-describe-)
    - [kubectl explain ](#kubectl-explain-)
    - [kubectl get ](#kubectl-get-)
    - [Change output format](#change-output-format)
    - [Declarative files from imperative commands](#declarative-files-from-imperative-commands)



## Configuration

Default configuration is `~/.kube/config`. The command `kubectl config view` shows that information.

Setting `kubectl --kubeconfig` allows to set a different config file.

Configuration contains kubernetes cluster IP (`--server` option) and path to TLS certificates (`--user` option).


The access configuration is stored in contexts. You can switch between different contexts. For example to permanently save the namespace for all subsequent kubectl commands in the current context: `kubectl config set-context --current --namespace=my-ns`

## Common commands

### kubectl run

`kubectl run` creates and runs a particular image in a pod. The `--dry-run=client` flag previews the command, without submitting it.

### kubectl apply

`kubectl apply` creates or updates resources. Use `apply` as the preferred way for working with "infrastructure as code" methodology.

### kubectl delete

`kubectl delete` deletes resources.

It is preferred to use it with a file for satisfying infrastructure as code, although it can also target resources by label selectors, resource type or names.

## Get information

### kubectl describe <resource>

This command needs a resource type or a resource name

`kubectl describe node <node-name>` or `kubectl describe node/<node-name>` are the same. It will show all information on resources running in the node.

### kubectl explain <type>

It gives basic information on resources. This changes depending on the distribution and also the add-ons installed.

List all sub-resources `--recursive`.

Lists available resources types `kubectl api-resources`.

A very similar information should be found in API official documentation.

### kubectl get <resource>

The most useful resources are:
- `nodes`: Lists information on all nodes in the cluster. Shortcut of `node` can be `no` or `nodes`.
- `services`: Service is a stable endpoint to connect to. Shortcut of `services` can be `svc`.
- `pods`: Lists pods in a specific namespace: `kubectl get pods -n <namespace-name>`. By default it uses the `default` namespace. To list absolutely all pods in cluster: `kubectl get pods --all-namespaces` or `kubectl get pods -A`
- `namespaces`: The namespaces logically split the resources in a cluster. Shortcut can be `ns` or `namespace`.


### Change output format

`kubectl [command] [TYPE] [NAME] -o <output_format>` where the output format can be changed to

- **-o name**: Only output the resource name.
- **-o wide**: Output in plain-text format with additional information.
- **-o yaml**: Output a YAML formatted API object.
- **-o json**: Output a JSON formatted API object.

### Declarative files from imperative commands

Definition files can easily be created with the imperative commands.

- `-dry-run=client` will not execute the command, instead it will check whether the command is valid and can be executed.
- `-o yaml` will print the command in yaml file format

For example:

```bash
$ kubectl run front-ui --image=nginx --dry-run=client -o yaml > pod-definition.yml
$ kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yml
$ kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml > service-definition.yml
```

This can be specially useful for creating a pod and exposing the pod in a service with one command:

```bash
$ kubectl run httpd --image=httpd:alpine --port 80 --expose --dry-run=client -o yaml > pod-service-definition.yml
```


The `pod-service-definition.yml` file will be:

```yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  name: httpd
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: httpd
status:
  loadBalancer: {}
---
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: httpd
  name: httpd
spec:
  containers:
  - image: httpd:alpine
    name: httpd
    ports:
    - containerPort: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
