# Networking

Networking is a critical component of Kubernetes, responsible for routing traffic within the cluster (inter-pod communication) and to the outside world (ingress and egress traffic).

The Kubernetes network model is implemented using the **Container Network Interface (CNI)** plugins, which are executed by the container runtime on each node.

## Container Network Interface (CNI)

CNI is a specification that defines how container runtimes interact with the network. CNI plugins implement this specification and are responsible for:

- Setting up network namespaces.
- Configuring network interfaces.
- Managing traffic routing for containers.

CNI plugins are invoked by the container runtime when a container is created or deleted. In Kubernetes, they enable pod networking and ensure seamless traffic routing between pods running across different nodes.


### Listing current CNI binaries

CNI binaries are typically located in the `/opt/cni/bin/` directory. You can verify the installed plugins using the following command:

```bash
$ ls /opt/cni/bin
bridge  dhcp  flannel  host-local  ipvlan  loopback  macvlan  portmap  ptp  tuning  vlan
```

Each binary corresponds to a specific CNI plugin or capability.

### CNI configuration file

CNI configuration files, usually located in `/etc/cni/net.d/`, define how the network is set up. Some systems might use `/opt/cni/net.d/` or a custom path defined during setup

These files specify:
- The plugin to use.
- The order of execution.
- Custom parameters like IP ranges and routing settings.

Example configuration (`10-weave.conflist`):

```bash
$ cat /etc/cni/net.d/10-weave.conflist 
{
    "cniVersion": "0.3.0",
    "name": "weave",
    "plugins": [
        {
            "name": "weave",
            "type": "weave-net",
            "hairpinMode": true
        },
        {
            "type": "portmap",
            "capabilities": {"portMappings": true},
            "snat": true
        }
    ]
}
```

In this example:

- The primary plugin (`weave-net`) handles pod-to-pod traffic.
- The `portmap` plugin manages port mappings for external traffic.

### Container Runtime Endpoint

CNI plugins are invoked by the container runtime through a specified endpoint. For example, in Kubernetes, the `kubelet` service is configured with a `--container-runtime-endpoint` flag that points to the container runtime's Unix socket. 

This endpoint allows the `kubelet` to communicate with the container runtime (e.g., `containerd` or `CRI-O`) to manage networking.

```bash
$ ps -ef | grep kubelet
/usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock ...
```

In the example above, the `kubelet` service is started with the `--container-runtime-endpoint` option to specify the CNI endpoint located at `/var/run/containerd/containerd.sock`.

Alternatively, the CNI endpoint can be specified in the CNI configuration file located in the `/etc/cni/net.d` directory.


## CNI plugins

The most popular CNI plugins are `weave-net`, `calico`, `flannel`, and `cilium`. Each CNI plugin has its own way of setting up the network and routing the traffic. 

CNI plugins can be broadly categorized based on their networking approaches:

### 1. Overlay Networks

These plugins create a virtual network on top of the existing physical network. Packets are encapsulated (wrapped) within another packet to travel across the underlying network.

For example, VXLAN (Virtual Extensible LAN) wraps a layer 2 Ethernet packet inside a UDP packet. So in essence it creates a UDP tunnel overlaying the existing underlying network to connect pods on different nodes.

- **Advantages:** Simplifies network setup as it doesn't require changes to the underlying network infrastructure.
- **Disadvantages:** Performance overhead due to encapsulation/decapsulation.

Examples:
- **Flannel (with VXLAN backend):** A popular and simple overlay network. It uses VXLAN (Virtual Extensible LAN) to encapsulate packets.
- **Calico (with IPIP or VXLAN):** Calico can operate in overlay mode using either IPIP (IP-in-IP tunneling) or VXLAN.
- **Weave Net:** Uses its own form of encapsulation or VXLAN and also provides features like network policy enforcement and encryption.

### 2. Layer 3 Routed Networks

These plugins integrate directly with the underlying physical network. Pods get IP addresses routable within the physical network.

- **Advantages:** Better performance compared to overlay networks (no encapsulation overhead).
- **Disadvantages:** Requires tighter integration with the underlying network, and may require changes to the physical network infrastructure (e.g., configuring BGP).

Examples:

- **Calico (with BGP):** Calico is well-known for its BGP (Border Gateway Protocol) mode, which allows it to advertise pod routes to the physical network routers, enabling direct routing.
- **Cilium (with BGP or direct routing):** Cilium can also use BGP or other direct routing methods, leveraging eBPF for efficient packet processing.
- **AWS VPC CNI:** Tightly integrated with AWS infrastructure, providing native VPC networking to pods. It directly assigns Elastic Network Interfaces (ENIs) to pods for underlay integration.
- **Azure CNI** Similar to AWS VPC CNI, it integrates with Azure's networking to provide native VNet connectivity to pods.
- **GCP CNI:** Google Cloud's CNI plugin that integrates with Google's Virtual Private Cloud (VPC) networking.

### 3. Layer 2 Networks:

These plugins rely on Layer 2 (data link layer) mechanisms to connect pods. Pods are typically on the same subnet.

- **Advantages:** Simple to set up in environments where Layer 2 connectivity is readily available.
- **Disadvantages:** Limited scalability as it relies on broadcast domains. Pods generally need to be on the same subnet.

Examples:

- **Flannel (with host-gw backend):** In host-gw mode, Flannel sets up routes on each host to reach pods on other hosts, but it relies on Layer 2 adjacency between nodes.
- **Calico (with Layer 2 mode):** Calico can also operate in a pure Layer 2 mode, although it's less common than its BGP or overlay modes.
- **Bridge CNI:** A simple CNI plugin that connects pods to a Linux bridge on the host.

### 4. eBPF-based Networks

These plugins leverage eBPF (extended Berkeley Packet Filter) for efficient packet processing and network security without the need for Linux kernel modules.

- **Advantages:** High performance, fine-grained control over network policies, and security. Enables sophisticated features like load balancing, network policy enforcement, and observability at a granular level.
- **Disadvantages:** Requires kernel support for eBPF. Implementing and managing eBPF programs can be more complex than simpler CNI plugins.

Examples:

- **Cilium:** A leading CNI plugin that heavily utilizes eBPF for networking, security, and observability.
- **Calico (with eBPF data plane):** Calico also offers an eBPF data plane option for improved performance and features.

## Weaver CNI plugin

As an example, let's look at the `weave-net` CNI plugin.

The routes and network configuration are managed by the `weave` pods running on the Kubernetes cluster.

```bash
$ kubectl get pods -n kube-system -o wide
NAME                                   READY   STATUS    RESTARTS      AGE   IP             NODE
kube-scheduler-controlplane            1/1     Running   0             40m   192.11.10.12   controlplane   
...
weave-net-bvcgw                        2/2     Running   1 (39m ago)   39m   192.11.10.12   controlplane   
weave-net-m9jbx                        2/2     Running   0             39m   192.11.10.3    node01         
```

The following command lists the virtual network interface created by the `weave-net` plugin.

```bash
$ ip addr show weave
4: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP group default qlen 1000
    link/ether 7a:18:b7:d6:5a:89 brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.1/16 brd 10.244.255.255 scope global weave
       valid_lft forever preferred_lft forever
```

In the example above, the `weave` network interface is created with the IP address range `10.244.x.x`.

Each node will have a default route to the `weave` network interface.

```bash
controlplane $ ip route show
default via 172.25.0.1 dev eth1 
10.244.0.0/16 dev weave proto kernel scope link src 10.244.0.1 
172.25.0.0/24 dev eth1 proto kernel scope link src 172.25.0.90 
192.11.10.0/24 dev eth0 proto kernel scope link src 192.11.10.12 

node01 $ ip route show
default via 172.25.0.1 dev eth1 
10.244.0.0/16 dev weave proto kernel scope link src 10.244.192.0 
172.25.0.0/24 dev eth1 proto kernel scope link src 172.25.0.81 
192.11.10.0/24 dev eth0 proto kernel scope link src 192.11.10.3 
```

In the example above, the virtual network interface routes the traffic through the `10.244.0.1` IP address. At the `node01`, the traffic is routed through the `10.244.192.0` instead.

