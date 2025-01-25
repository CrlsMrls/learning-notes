# Networking Discovery Commands

- [Networking](#networking)
  - [ip command](#ip-command)
    - [Show network interfaces](#show-network-interfaces)
    - [Show IP addresses](#show-ip-addresses)
    - [Show routing table](#show-routing-table)
  - [netstat command](#netstat-command)
    - [Show services listening on network connections](#show-services-listening-on-network-connections)
    - [Show network statistics](#show-network-statistics)
    - [Show all active network connections](#show-all-active-network-connections)
    - [Show routing table](#show-routing-table-1)
  - [arp command](#arp-command)
    - [list all devices in the ARP table](#list-all-devices-in-the-arp-table)


## ip command

`ip` is a powerful tool for configuring network interfaces. It is part of the `iproute2` package.

### Show network interfaces

```bash
$ ip link

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: eth0@if36154: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 42:7a:86:eb:ba:29 brd ff:ff:ff:ff:ff:ff link-netnsid 0
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UNKNOWN mode DEFAULT group default 
    link/ether d6:6f:c4:76:4e:04 brd ff:ff:ff:ff:ff:ff
5: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 1e:89:5e:69:fa:65 brd ff:ff:ff:ff:ff:ff
6: veth6cef9fd6@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue master cni0 state UP mode DEFAULT group default 
    link/ether 1e:7f:bd:47:0c:6d brd ff:ff:ff:ff:ff:ff link-netns cni-9043b80f-a32a-d846-4e0a-f837def529b0
7: veth8d8e37aa@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue master cni0 state UP mode DEFAULT group default 
    link/ether de:57:aa:18:d9:fe brd ff:ff:ff:ff:ff:ff link-netns cni-866e8acc-ac22-e5bb-385f-563249095383
```

In the previous example, `lo` is the loopback interface, `eth0` is the ethernet interface, `flannel.1` is the flannel interface, and `cni0` is the cni interface. 

Containerd creates a bridge interface for all containers, in this case, `cni0`. The `veth` interfaces are the virtual ethernet interfaces that connect the containers to the bridge.

To show only the interfaces that are up:

```bash
$ ip link show up
```

And to focus on a specific interface:

```bash
$ ip link show cni0
5: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 1e:89:5e:69:fa:65 brd ff:ff:ff:ff:ff:ff
```

In the previous example `BROADCAST,MULTICAST,UP,LOWER_UP` means that the interface is up and running. `mtu` is the maximum transmission unit, `qdisc` is the queuing discipline, `state` is the state of the interface, `mode` is the mode of the interface, `group` is the group of the interface, and `qlen` is the queue length.

### Show IP addresses

```bash
$ ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
3: eth0@if36154: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 42:7a:86:eb:ba:29 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.233.171/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::407a:86ff:feeb:ba29/64 scope link 
       valid_lft forever preferred_lft forever
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UNKNOWN group default 
    link/ether d6:6f:c4:76:4e:04 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::d46f:c4ff:fe76:4e04/64 scope link 
       valid_lft forever preferred_lft forever
5: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1360 qdisc noqueue state UP group default qlen 1000
    link/ether 1e:89:5e:69:fa:65 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/24 brd 172.17.0.255 scope global cni0
       valid_lft forever preferred_lft forever
    inet6 fe80::1c89:5eff:fe69:fa65/64 scope link 
       valid_lft forever preferred_lft forever
```

To focus on an ip address:

```bash
 ip a | grep -B2 192.168.233.171
3: eth0@if36154: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default 
    link/ether 42:7a:86:eb:ba:29 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.233.171/32 scope global eth0
```

### Show routing table

```bash
$ ip route
default via 169.254.1.1 dev eth0 
169.254.1.1 dev eth0 scope link 
172.17.0.0/24 dev cni0 proto kernel scope link src 172.17.0.1 
172.17.1.0/24 via 172.17.1.0 dev flannel.1 onlink 
```

In the previous example, `default` means the default route, `via` is the gateway, `dev` is the device, `scope` is the scope of the route, `link` is the link, `src` is the source, and `proto` is the protocol.

## netstat command

`netstat` is a command-line tool that displays network connections, routing tables, and a number of network interface statistics.

### Show services listening on network connections 

- `n` shows the numerical addresses.
- `p` shows the process ID and the name of the program to which each socket belongs.
- `t` shows the TCP connections.
- `l` shows only listening sockets.

```bash
$ netstat -nptl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      543/systemd-resolve 
tcp        0      0 192.168.233.171:2380    0.0.0.0:*               LISTEN      3648/etcd           
tcp        0      0 192.168.233.171:2379    0.0.0.0:*               LISTEN      3648/etcd           
tcp        0      0 127.0.0.1:44387         0.0.0.0:*               LISTEN      954/containerd      
tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      3854/kube-scheduler 
tcp        0      0 127.0.0.1:10257         0.0.0.0:*               LISTEN      3343/kube-controlle 
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      4627/kube-proxy     
tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      4123/kubelet        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      956/sshd: /usr/sbin 
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      958/ttyd            
tcp        0      0 127.0.0.1:2381          0.0.0.0:*               LISTEN      3648/etcd           
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      3648/etcd           
tcp6       0      0 :::8888                 :::*                    LISTEN      4304/kubectl        
tcp6       0      0 :::10256                :::*                    LISTEN      4627/kube-proxy     
tcp6       0      0 :::10250                :::*                    LISTEN      4123/kubelet        
tcp6       0      0 :::22                   :::*                    LISTEN      956/sshd: /usr/sbin 
tcp6       0      0 :::6443                 :::*                    LISTEN      3326/kube-apiserver 
```

In the previous example, `containerd` service is listening on port `44387`, `etcd` is listening on ports `2380` and `2379`, etc.

### Show network statistics

```bash
$ netstat -s
IcmpMsg:
    InType8: 129
    OutType0: 129
Tcp:
    3204 active connection openings
    3106 passive connection openings
    85 failed connection attempts
    1452 connection resets received
    144 connections established
    181938 segments received
    180642 segments sent out
    36 segments retransmitted
    0 bad segments received
    1564 resets sent
...
```

### Show all active network connections

- `a` shows all connections.
- `n` shows the numerical addresses.
- `p` shows the process ID and the name of the program to which each socket belongs.

```bash
$ netstat -anp | grep init
unix  3      [ ]         DGRAM      CONNECTED     174828764 1/init               /run/systemd/notify
unix  2      [ ]         DGRAM                    174828765 1/init               /run/systemd/cgroups-agent
unix  2      [ ACC ]     STREAM     LISTENING     174828768 1/init               /run/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     174828770 1/init               /run/systemd/userdb/io.systemd.DynamicUser
unix  2      [ ACC ]     STREAM     LISTENING     174828771 1/init               /run/systemd/io.system.ManagedOOM
unix  7      [ ]         DGRAM      CONNECTED     174812085 1/init               /run/systemd/journal/dev-log
unix  7      [ ]         DGRAM      CONNECTED     174812087 1/init               /run/systemd/journal/socket
unix  2      [ ACC ]     STREAM     LISTENING     174812089 1/init               /run/systemd/journal/stdout
unix  2      [ ACC ]     SEQPACKET  LISTENING     174812091 1/init               /run/udev/control
```

In the previous example, `init` is the first process that is started by the kernel. It is the parent of all other processes. These are all the connections that `init` has.

### Show routing table

Similar to the `ip route` command, `netstat -r` shows the routing table.

```bash
$ netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         169.254.1.1     0.0.0.0         UG        0 0          0 eth0
169.254.1.1     0.0.0.0         255.255.255.255 UH        0 0          0 eth0
172.17.0.0      0.0.0.0         255.255.255.0   U         0 0          0 cni0
172.17.1.0      172.17.1.0      255.255.255.0   UG        0 0          0 flannel.1
```

## arp command

`arp` is used to display and modify the Address Resolution Protocol (ARP) cache.

### list all devices in the ARP table

```bash
$ arp -a
? (172.17.1.0) at 1a:2b:71:88:bb:84 [ether] PERM on flannel.1
? (169.254.1.1) at ee:ee:ee:ee:ee:ee [ether] on eth0
? (172.17.0.2) at 12:de:bc:c1:cc:93 [ether] on cni0
? (172.17.0.3) at 26:b0:62:76:46:e3 [ether] on cni0
```

