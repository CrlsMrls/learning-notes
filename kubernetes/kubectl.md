# kubectl

kubectl is the command line tool for communicating with a Kubernetes cluster, it connects to control pane's API.

## Configuration

Default configuration is ~/.kube/config.

Setting `kubectl --kubeconfig` allows to set a different config file.

Configuration contains kubernetes cluster IP (`--server` option) and path to TLS certificates (`--user` option).

## Common commands

### kubectl run

`kubectl run` creates and runs a particular image in a pod. The `--dry-run=client` flag previews the command, without submitting it.

### kubectl apply

`kubectl apply` creates or updates resources. It is prefered to use `apply` for satisfying infrastructure as code.

### kubectl delete

`kubectl delete` deletes resources.

It is preferred to use it with a file for satisfying infrastructure as code, although it can also target resources by label selectors, resource type or names.

## Get information

### Change output format

Output format can be changed to

- -o wide
- -o yaml
- -o json

### kubectl describe <resource>

This command needs a resource type or a resource name

`kubectl describe node <node-name>` or `kubectl describe node/<node-name>` are the same. It will show all information on resources running in the node.

### kubectl explain <type>

It gives basic information on resources. This changes depending on the distribution and also the add-ons installed.

List all sub-resources `--recursive`.

Lists available resources types `kubectl api-resources`.

A very similar information should be found in API official documentation.

### kubectl get <resource>

#### kubectl get nodes

Lists information on all nodes in the cluster.

Shortcut of `node` can be `no` or `nodes`

#### kubectl get services

Shortcut of `services` can be `svc`

Service is a stable endpoint to connect to "something". Originally called "portals".

#### kubectl get pods

One of the most useful commands. By default it uses the `default` namespace.

List absolutely all pods in cluster: `kubectl get pods --all-namespaces` or `kubectl get pods -A`

Lists pods in a specific namespace: `kubectl get pods -n <namespace-name>`

#### kubectl get namespaces

Lists the namespaces. The namespaces logically split the resources in a cluster.

Shortcut can be `ns` or `namespace`.

Creates a new namespace: `kubectl create --namespace=<namespace-name>`
