# Resource Quota

Resource Quotas are a tool for administrators to control the amount of resources (CPU, memory, etc.) that can be consumed in a namespace. 

The `ResourceQuota` admission controller must be enabled in the environment, this is done by editing the `/etc/kubernetes/manifests/kube-apiserver.yaml` file and passing the command `- --enable-admission-plugins=NodeRestriction,ResourceQuota` to the container.

Although the `ResourceQuota` admission controller is enabled by default in most Kubernetes distributions, it is a good practice to verify that it is enabled in your cluster by running the following command:

```bash
$ kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'enable-admission-plugins' | grep ResourceQuota
  --enable-admission-plugins strings  admission plugins that should be enabled in addition to default enabled ones (NamespaceLifecycle, LimitRanger, ServiceAccount, ... ResourceQuota, ...).
```

## Creating a Resource Quota

The following is an example of a `ResourceQuota` object:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-a-resource-quota
  namespace: team-a
spec:
  hard:
    requests.cpu: "0.5"
    requests.memory: "500Mi"
    limits.cpu: "1"
    limits.memory: "1G"
    memory: "1Gi"
    pods: "5"
```

The previous example creates a `ResourceQuota` object named `team-a-resource-quota` in the `team-a` namespace, specifying the limits for CPU, memory, and the number of pods that can be created in the namespace.

## Describing Resource Quotas

```bash
$ kubectl describe resourcequotas -n team-a team-a-resource-quota 
Name:            team-a-resource-quota
Namespace:       team-a
Resource         Used  Hard
--------         ----  ----
limits.cpu       0     1
limits.memory    0     1G
memory           0     1Gi
pods             0     5
requests.cpu     0     500m
requests.memory  0     500Mi
```

## Failure to Create a Pod

If a pod is created in a namespace that violates the `ResourceQuota` object, the pod creation fails with the following error:

```bash
$ kubectl apply -f pod.yaml
Error from server (Forbidden): error when creating "pod.yaml": pods "app-pod" is forbidden: failed quota: team-a-resource-quota: must specify limits.cpu for: app-container; limits.memory for: app-container; memory for: app-container; requests.cpu for: app-container; requests.memory for: app-container
```

For resolving the error, the pod definition must include the resource limits and requests for the container:

```yaml
template:
   metadata:
     namespace: team-a
   spec:
     containers:
     - image: nginx:stable
       name: nginx
       ports:
       - containerPort: 80
         protocol: TCP
       resources:
          requests:
            cpu: "0.1"
            memory: "10Mi"
          limits:
            cpu: "0.2"
            memory: "50Mi"
```

After adding the resource limits and requests to the pod definition, the pod is created successfully.

```bash
$ kubectl describe resourcequotas -n team-a team-a-resource-quota 
Name:            team-a-resource-quota
Namespace:       team-a
Resource         Used      Hard
--------         ----      ----
limits.cpu       200m      1
limits.memory    52428800  1G
memory           10Mi      1Gi
pods             1         5
requests.cpu     100m      500m
requests.memory  10Mi      500Mi
```

More information about the `ResourceQuota` admission controller can be found in the [official documentation](https://kubernetes.io/docs/concepts/policy/resource-quotas/).

