# Volumes

Containers are ephemeral, they can be created and destroyed at any time. When a container is destroyed, all data in the container is lost. Volumes provide a way to store data outside of the container, so that it can be reused by other containers. They can be used to share data between containers in a Pod, or to share data between containers in different Pods.

The **Container Storage Interface (CSI)** is a standard for exposing arbitrary block and file storage systems. Currently, plugins are compiled into the `kubelet` binary, which makes it difficult to deploy and update plugins. CSI allows storage vendors to write plugins that can be dynamically deployed and updated.

- [Volumes](#volumes)
  - [Pod volumes](#pod-volumes)
  - [Volume types](#volume-types)
    - [1. EmptyDir](#1-emptydir)
    - [2. HostPath](#2-hostpath)
    - [3. ConfigMap](#3-configmap)
    - [4. Secret](#4-secret)
    - [5. Persistent Volumes and Claims](#5-persistent-volumes-and-claims)
    - [6. NFS and iSCSI](#6-nfs-and-iscsi)
    - [7. Cloud storage](#7-cloud-storage)
  - [References](#references)


## Pod volumes

- A pod specification can define one or more volumes in the `spec.volumes`. The `spec.volumes` field in the Pod spec defines details about volumes used in the Pod.
- Each volume can be mounted in one or more containers in the Pod. The `spec.containers.<container>.volumeMounts` field in the container spec mounts a volume to a specific container's filesystem at a specific location.
- The same volume can be mounted in multiple containers. There is nothing preventing multiple containers from overwriting each other's data in the volume, locking or versioning the data is the responsibility of the application. To prevent data corruption, the containers should not write to the same file in the volume.

## Volume types

There are several volume types, each with their pros and cons. The most important types are:

### 1. EmptyDir

`emptyDir` volumes provide temporary storage that uses the host file system and are removed when Pod is deleted. The kubelet will create the directory in the container, but not mount any storage. As a result, the initial directory is empty. 

The `emptyDir` volumes is normally used to share files between containers in a Pod.

The data in an `emptyDir` volume is safe across container restarts (container crashes), but is lost when the Pod is deleted or the node is rebooted. 

The `emptyDir` includes the following fields:
- `Medium` - The `Medium` field defines the type of storage (disk, SSD, network storage, etc.) . The default is `""`, which uses the node's default storage medium. The other options are `Memory` (for a `tmpfs` RAM-backed filesystem) and `HugePages`.
- `SizeLimit` - The `SizeLimit` field defines the maximum size of the volume. The default is `nil`, which means that the volume can be as large as the node's free disk space. The other option is a size limit, such as `1Gi` or `100Mi`.

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: busybox
    containers:
spec:
    - image: busybox
      name: busy
      volumeMounts:
      - mountPath: /scratch
        name: scratch-volume # must match the volume name below
    volumes:
    - name: scratch-volume # must match the volume name above
      emptyDir: {}
```

### 2. HostPath

`hostPath` volumes mount data from a specific location on the host (k8s node). 

Avoid using `hostPath` volumes whenever possible.
- The recommendation is to use `hostPath` volumes only for development and testing. The `hostPath` volumes are **not suitable for production** use because they are tied to a specific node, and the data is not replicated across nodes. If the node fails, the data is lost.
- They present many **security risks**, because they allow containers to access the host filesystem and devices. They can be used to access sensitive files, such as `/etc/shadow` or `/etc/sudoers`, or to access devices, such as `/dev/sda` or `/dev/kvm`. They can also be used to access the host network, which can be used to bypass network security controls. 

The most important `hostPath` types can be:
- `Directory` - Mounts an existing directory on the host.
- `DirectoryOrCreate` - Mounts a directory on the host, and creates it if it does not already exist.
- `File` - Mounts an existing single file on the host.
- `FileOrCreate` - Mounts a file on the host, and creates it if it does not exist.
- `Socket` - Mounts an existing socket on the host.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - image: busybox
    name: test-container
    volumeMounts:
    - mountPath: /test-pd # mount path in the container
      name: test-volume # must match the volume name below
    - mountPath: /opt/1.txt
      name: myfile
  volumes:
  - name: test-volume # must match the volume name above
    hostPath:
      path: /data # path of the directory on the host 
      type: DirectoryOrCreate # optional - defaults to Directory if not specified
  - name: myfile
    hostPath:
      path: /var/local/aaa/1.txt
      type: FileOrCreate
```

### 3. ConfigMap

ConfigMaps allow to store configuration data in key-value pairs. 

Check the [ConfigMap](./13-configmaps.md) document.

### 4. Secret

Secrets objects allow to store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys. 

Check the [Secret](./14-secrets.md) document.

### 5. Persistent Volumes and Claims

The `PersistentVolume` (PV) and `PersistentVolumeClaim` (PVC) objects provide a way to use persistent storage with Kubernetes. 

Check the [Persistent Volumes and Claims](./15-persistent-volumes.md) document.

### 6. NFS and iSCSI

NFS (Network File System) and iSCSI (Internet Small Computer System Interface - SCSI over IP) are straightforward choices for multiple readers scenarios.

- A NFS volume allows an existing NFS (Network File System) share to be mounted into a Pod. NFS can be mounted by multiple writers simultaneously.
- An iSCSI volume allows an existing iSCSI (Internet Small Computer System Interface) volume to be mounted into a Pod. Unfortunately, iSCSI volumes can only be mounted by a single consumer in read-write mode. 

NFS and iSCSI servers must be running with the volume created before use it.

### 7. Cloud storage

Cloud storage providers, such as AWS EBS, Azure Disk, and Google Cloud Persistent Disk, provide durable, high-performance block storage for applications running in the cloud.

## References

- [Kubernetes Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)
- [AWS EBS](https://aws.amazon.com/ebs/)
- [Azure Disk](https://azure.microsoft.com/en-us/services/storage/disks/)
- [Google Cloud Persistent Disk](https://cloud.google.com/persistent-disk/)
