# CoreDNS

[CoreDNS](https://coredns.io/) is a flexible, extensible DNS server that can serve as the Kubernetes cluster DNS. 

## CoreDNS Architecture

CoreDNS acts as the cluster's DNS service, it is deployed as a set of pods in the `kube-system` namespace as a `kube-dns` service. 

Inside the pod, DNS request are forwarded to the upstream DNS server specified in the `/etc/resolv.conf` file pointing to the `kube-dns` service IP address, which is the ClusterIP of the CoreDNS service.

## CoreDNS Service

The `kube-dns` service forwards DNS requests to the `coredns` pods. It exposes port `53` for both `UDP` and `TCP` traffic to handle DNS queries.


```bash
$ kubectl get svc -n kube-system
NAME               TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   172.20.0.10   <none>        53/UDP,53/TCP,9153/TCP   11m
```

## CoreDNS Pods

The `coredns` pods are responsible for handling DNS requests. These pods are deployed as part of the `kube-dns` service in the `kube-system` namespace. They listen for DNS queries within the cluster.

```bash
$ kubectl get pods -n kube-system 
NAME                                       READY   STATUS    RESTARTS   AGE
pod/coredns-77d6fd4654-4k26m               1/1     Running   0          8m46s
pod/coredns-77d6fd4654-jwd4d               1/1     Running   0          8m46s
...
```


## CoreDNS Configuration

CoreDNS uses a `ConfigMap` named `coredns` in the `kube-system` namespace to manage its configuration. This `ConfigMap` contains the `Corefile` that specifies DNS zones and plugins.

```bash
$ kubectl get cm coredns -n kube-system -o yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health {
           lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           fallthrough in-addr.arpa ip6.arpa
           ttl 30
        }
        prometheus :9153
        forward . /etc/resolv.conf {
           max_concurrent 1000
        }
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
...
```

The following plugins are used in the `Corefile` configuration:
- **kubernetes:** Resolves Kubernetes service and pod names to IP addresses.
- **forward:** Forwards unresolved DNS queries to upstream DNS servers specified in /etc/resolv.conf.
- **cache:** Caches DNS responses to improve performance.
- **health:** Provides an endpoint to check the health of CoreDNS.
- **prometheus:** Exposes CoreDNS metrics for Prometheus.
- **loadbalance:** Balances DNS query load across upstream servers.

This configuration is mounted as a volume in the `coredns` pods at `/etc/coredns/Corefile`. To verify the volume mount, use: `kubectl get deployments.apps coredns -n kube-system -o yaml | grep -B3 Corefile`

## DNS Configuration in Pods

The `/etc/resolv.conf` file in each pod contains the IP address of the `kube-dns` service. This file ensures pods can resolve both internal and external domains.

```bash
$ kubectl exec -it test -- cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 172.20.0.10
options ndots:5
```

- **nameserver:** Points to the `kube-dns` service IP. If this entry is missing or incorrect, DNS resolution within the cluster will fail.
- **options ndots:5:** Ensures that DNS queries for internal services are properly resolved.


## Checking DNS resolution in Pods

To validate DNS resolution within a pod, use tools like `ping`, `curl`, `nslookup`, or `dig` from within the pod.

```bash
$ kubectl exec -it app -- ping -c 4 google.com
$ kubectl exec -it app -- curl http://service.namespace.svc.cluster.local
$ kubectl exec -it app -- nslookup service.namespace.svc.cluster.local
```

For example, for the service name (e.g., `web-service`) and namespace (e.g., `default`) in the format `service.namespace.svc.cluster.local`. To ensure the service resolves correctly:

```bash
$ kubectl get svc web-service
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
web-service   ClusterIP   172.20.71.52   <none>        80/TCP    33m

$ kubectl exec -it app -- nslookup web-service
Server:         172.20.0.10
Address:        172.20.0.10#53

Name:   web-service.default.svc.cluster.local
Address: 172.20.71.52
```

This command verifies that the `web-service` name resolves to the correct IP address within the pod.

## Debugging DNS Issues:

1. Check DNS Resolution: Use `nslookup` or `dig` to verify DNS resolution within the pod.
2. Check CoreDNS Logs: `kubectl logs -n kube-system -l k8s-app=kube-dns`
3. Test Connectivity to Upstream DNS: Verify if CoreDNS pods can reach the upstream DNS server (e.g., Google DNS 8.8.8.8).
4. Verify ConfigMap: Ensure the `Corefile` is correctly configured, particularly the kubernetes and forward plugins.
5. Check Network Policies: Ensure that NetworkPolicies allow traffic between pods and the kube-dns service.
6. Verify Pod DNS Configuration: Check the `/etc/resolv.conf` file in the pod to ensure it points to the correct `kube-dns` service IP.
  