# gVisor

gVisor (https://gvisor.dev/) is container runtime sandbox that provides an additional layer of security for running containers. It is maintaned by Google and implements a security boundary between the application and the host kernel by intercepting application system calls and acting as a guest kernel, rather than allowing direct access to the host kernel.

The main features of gVisor are:

- Enhanced isolation: Reduces the attack surface of the kernel
- Compatible with OCI (Open Container Initiative) containers
- Lightweight compared to VMs but provides similar security benefits
- Designed specifically for multi-tenant environments

gVisor is not a syscaller proxy like `seccomp` or `AppArmor`, neither a full-blown VM. It is a full-blown kernel that runs in user space and intercepts system calls from the application. This allows gVisor to provide a more secure environment for running containers.

## runsc

`runsc` is the gVisor runtime that runs containers in a sandboxed environment. It is a lightweight, OCI-compatible runtime that can be used with containerd or Docker to run containers with gVisor.

## Running Pods with gVisor in Kubernetes

To run pods with gVisor in Kubernetes, you need to:

- Install gVisor (runsc) on your nodes
- Configure containerd to use gVisor as a runtime
- Create a `RuntimeClass` resource
- Reference the `RuntimeClass` in your pod specifications
- Verify that the pod is running with gVisor

### 1. Install gVisor on your cluster nodes

Refer to the [official documentation](https://gvisor.dev/docs/user_guide/install/) for instructions on how to install gVisor on your nodes.

### 2. Configure containerd to use gVisor as a runtime

Edit the containerd configuration file (`/etc/containerd/config.toml`) and add the following configuration:

```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runsc]
  runtime_type = "io.containerd.runsc.v1"
```

Restart containerd:

```bash
sudo systemctl restart containerd
```

### 3. Create a RuntimeClass resource

Create a `RuntimeClass` resource that specifies the gVisor runtime:

```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  # The name the RuntimeClass will be referenced by.
  name: gvisor 
# The name of the corresponding CRI configuration
handler: runsc
``` 

`RuntimeClass` is a non-namespaced resource. Apply the configuration using `kubectl apply -f runtimeclass.yaml`.

### 4. Reference the RuntimeClass in your pod specifications

In your pod specifications, add the `runtimeClassName` field to specify the `RuntimeClass` to use:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-gvisor
spec:
  runtimeClassName: gvisor
  containers:
  - name: nginx
    image: nginx
```

### 5. Verify that the pod is running with gVisor

To verify that the pod is running with gVisor, exec `dmesg` in the pod and look for the gVisor startup messages:

```bash
$ kubectl exec pods/gvisor-test -it -- dmesg
[    0.000000] Starting gVisor...
[    0.205840] Committing treasure map to memory...
[    0.541163] Granting licence to kill(2)...
[    0.885004] Searching for socket adapter...
[    0.938340] Verifying that no non-zero bytes made their way into /dev/zero...
[    1.421814] Checking naughty and nice process list...
[    1.662659] Creating bureaucratic processes...
[    2.002715] Searching for needles in stacks...
[    2.128826] Forking spaghetti code...
[    2.443890] Gathering forks...
[    2.783704] Digging up root...
[    3.279403] Setting up VFS...
[    3.653644] Setting up FUSE...
[    3.815482] Ready!
```

This confirms that the pod is running with gVisor.
