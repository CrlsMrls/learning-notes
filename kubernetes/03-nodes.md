# Nodes

A node may be a virtual or physical machine, depending on the cluster. Each node is managed by the control plane and contains the services necessary to run Pods: `kubelet`, `kube-proxy` and a container runtime.

Aliases: `nodes` and `no`

- [Nodes](#nodes)
  - [Get information on nodes](#get-information-on-nodes)


## Get information on nodes

```bash
$ kubectl get nodes
NAME                  STATUS     ROLES         AGE     VERSION
01.my-server.com    Ready      control-plane   4d23h   v1.28.1
02.my-server.com    Ready      worker1         4d3h    v1.28.1
03.my-server.com    Ready      worker2         4d2h    v1.28.1
04.my-server.com    NotReady   worker3         4d2h    v1.28.1
```

For getting environment and running OS:

```bash
$ kubectl get nodes -o wide
kubectl get nodes -o wide
NAME                 STATUS  ROLES        AGE     VERSION   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
01.my-server.com   Ready   control-plane  4d23h   v1.28.1   Ubuntu 20.04.6 LTS   5.15.0-1043-aws   containerd://1.6.22
02.my-server.com   Ready   worker1        4d3h    v1.28.1   Ubuntu 20.04.6 LTS   5.15.0-1043-aws   containerd://1.6.22
03.my-server.com   Ready   worker2        4d2h    v1.28.1   Ubuntu 20.04.6 LTS   5.15.0-1043-aws   containerd://1.6.22
04.my-server.com   Ready   worker3        4d2h    v1.28.1   Ubuntu 22.04.3 LTS   6.2.0-1010-aws    containerd://1.6.22
```