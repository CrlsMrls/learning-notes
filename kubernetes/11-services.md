# Services

In Kubernetes, the `Service` object is an abstraction layer how to connect to a Pod. It provides a single IP address and DNS name by which the Pod can be accessed reliably, regardless of where it is scheduled in the cluster.

Pods are ephemeral, they can be created and destroyed at any time. When a Pod is destroyed, its IP address is released and can be reused by another Pod. This means that the IP address of a Pod is not reliable. Services, on the other hand, are long-lived objects that have a stable IP address and DNS name.

Depending on the Service type, the Service can be accessed from inside the cluster, from outside the cluster, or both.

Aliases: `service` or `services`




## Service types

There are four types of Services:
1. `ClusterIP` - The default type and only accessible from inside the cluster.
2. `NodePort` - Accessible from outside the cluster using `<NodeIP>:<NodePort>`.
3. `LoadBalancer` - Accessible from outside the cluster using a cloud provider's load balancer.
4. `ExternalName` - The Service maps a DNS to external URLs.

### 1. Service type ClusterIP

The `ClusterIP` type is the default Service type. 

- This is the most basic and most commonly used type of Service. 
- It provides a **stable IP address, port number, and DNS name** to access Pods. Facilitating service discovery and load balancing.
- This Service is only **accessible from inside** the cluster as it is exposed on a cluster-internal IP address from the CIDR range. 

```yaml
apiVersion: v1
kind: Service
metadata:
    name: service-name
spec:
    type: ClusterIP # optional, as it is the default value
    selector:
        app: some-label # the selector must match the Pod labels
    ports:
    - port: 80 # the port exposed by the Service
      targetPort: 8080 # the port exposed by the Pod
```

Pods in different namespaces can access using the DNS name of the Service: `<service-name>.<namespace>.svc.cluster.local`. Pods in the same namespace can access using the DNS name of the Service: `<service-name>`.

The `.spec.clusterIP` field allows to define an IP address. The default is `None`, which means that the Service is not assigned an IP address.


### 2. Service type NodePort

This Service is accessible from outside the cluster using `<NodeIP>:<NodePort>`.

The `NodePort` type is a superset of the `ClusterIP` type. It provides the same functionality as the `ClusterIP` type, but also exposes the Service on a port on each node of the cluster.

It is not recommended to use the `NodePort` type for production, as there are some drawbacks:
- It exposes the Service on a port on each node of the cluster, which is a security risk.
- It requires the user to know the IP address of a node, which is not always possible.
- All nodes must have the same available port. This means that the port cannot be used by another Service. 

```yaml
apiVersion: v1
kind: Service
metadata:
    name: service-name
spec:
    type: NodePort
    selector:
        app: some-label # the selector must match the Pod labels
    ports:
    - port: 80 # the port exposed by the Service
      targetPort: 8080 # the port exposed by the Pod
      nodePort: 30080 # the port exposed on each node (30000-32767)
```
 
### 3. Service type LoadBalancer

This Service is accessible from outside the cluster using a cloud provider's load balancer.

The `LoadBalancer` type is a superset of the `NodePort` type. It provides the same functionality as the `NodePort` type, but also provisions a cloud provider's load balancer to expose the Service.

```yaml
apiVersion: v1
kind: Service
metadata:
    name: service-name
spec:
    type: LoadBalancer
    selector:
        app: some-label # the selector must match the Pod labels
    ports:
    - port: 80 # the port exposed by the Service
      targetPort: 8080 # the port exposed by the Pod
      nodePort: 30080 # the port exposed on each node (30000-32767)
```

### 4. Service type ExternalName

The `ExternalName` type is a special type of Service that maps a DNS name to external URLs. It does not expose any ports or IP addresses. It is useful when migrating an application to Kubernetes, as it allows to use the old DNS name.

```yaml
apiVersion: v1
kind: Service
metadata:
    name: service-name
spec:
    type: ExternalName
    externalName: my.database.example.com # the external URL
```

In the previous example, the DNS will translate `<service-name>.<namespace>.svc.cluster.local` to `my.database.example.com`.


## Other configuration options

### Session affinity

The `sessionAffinity` field defines how the Service routes traffic to Pods when there are multiple Pods for a Service. The default is `None`, which means that the Service uses round-robin load balancing. The other option is `ClientIP`, which means that the Service uses the client's IP address to determine which Pod to use. This is useful for sticky sessions.

### Named ports

Ports can use name instead of numbers:

```yaml
apiVersion: v1
kind: Service
(...)
spec:
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```

In the Pod, the port name must match the targetPort name:

```yaml
apiVersion: v1
kind: Pod
(...)
spec:
  containers:
  - name: http-web
    image: nginx
    ports:
    - name: http-web-svc # must match the targetPort name in the Service
      containerPort: 80
```

This is useful to add semantics to the ports, which offers flexibility when the ports change.

### Multi-port Services

A Service can expose multiple ports:

```yaml
apiVersion: v1
kind: Service
(...)
ports:
- name: http
  protocol: TCP
  port: 80
  targetPort: http-web-svc
- name: https
  protocol: TCP
  port: 443
  targetPort: https-web-svc
```

This is useful when the application uses multiple ports.

****
## References

- [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)

