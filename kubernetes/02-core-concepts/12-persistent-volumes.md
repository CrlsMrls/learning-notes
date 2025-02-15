# Persistent Volumes and Claims

The `PersistentVolume` (PV) and `PersistentVolumeClaim` (PVC) objects provide a way to use persistent storage with Kubernetes. Developers do not create PV directly, instead they create PVC objects that request storage resources. The PV are bound to PVC that meet the requirements criteria. 

A PV represents the actual storage volume, the PVC represents the request for storage and a Pod can later use the PVC as a volume.

Most of the time the persistent disks are provisioned dynamically using [Storage Classes](#dynamic-provisioning) instead of creating the PV manually. 

- [Persistent Volumes and Claims](#persistent-volumes-and-claims)
  - [Access mode](#access-mode)
  - [Persistent volumes (PV)](#persistent-volumes-pv)
    - [YAML Definition of a PV](#yaml-definition-of-a-pv)
    - [Creation of a PV](#creation-of-a-pv)
  - [Persistent volume claims (PVC)](#persistent-volume-claims-pvc)
    - [YAML Definition of a PVC](#yaml-definition-of-a-pvc)
    - [Creation of a PVC](#creation-of-a-pvc)
    - [Assign a PVC to a Pod](#assign-a-pvc-to-a-pod)
  - [Dynamic provisioning with Storage Classes](#dynamic-provisioning-with-storage-classes)
    - [YAML Definition of a StorageClass](#yaml-definition-of-a-storageclass)
    - [Creation of a StorageClass](#creation-of-a-storageclass)
    - [PVC with a StorageClass](#pvc-with-a-storageclass)
    - [PVC with a StorageClass and a volume binding mode](#pvc-with-a-storageclass-and-a-volume-binding-mode)
  - [Common issues](#common-issues)
    - [PVC does not bound with a PV](#pvc-does-not-bound-with-a-pv)
    - [Deleting a PVC](#deleting-a-pvc)
  - [References](#references)

## Access mode

The `PersistentVolume` resource has a `spec.accessModes` field that specifies what modes the volume can be mounted with. This allows the cluster administrator to control how the volume can be used. These are the available modes:
- `ReadWriteOnce` or `RWO` - The volume can be mounted as read-write by a single node, this is the only mode that can be used with `hostPath` volumes.
- `ReadOnlyMany` or `ROX` - The volume can be mounted read-only by many nodes and many volume types, but is rarely used.
- `ReadWriteMany` or `RWX` - The volume can be mounted as read-write by many nodes. It is only supported by a few volume types, such as NFS and GlusterFS.
- `ReadWriteOncePod` or `RWOPod` - The volume can be mounted as read-write by a single pod. It is only supported by a few volume types, such as `hostPath` and `local`.

Not all volume types support all access modes. For example, `hostPath` volumes only support `ReadWriteOnce` and `ReadWriteOncePod` modes.

The `PersistentVolumeClaim` must match the `spec.accessModes` with the PV in order to bound them.

## Persistent volumes (PV)

`PersistentVolume` (PV) defines an abstract storage resource available for later consumption. It defines the type and amount of storage needed for the application. The PV is a cluster resource, so it can be used by any application in the cluster, but it can only be used by one application at a time and the namespace of the PV must match the namespace of the PVC.

The `PersistentVolume` (PV) objects are cluster-wide resources like a node is a cluster resource. 

### YAML Definition of a PV
The PV yaml file must have the following fields:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hostdata-pv
spec:
  capacity:
    storage: 1Gi 
  accessModes:
  - ReadWriteOnce 
  storageClassName: host
  hostPath:
    path: /host/voldata
    type: Directory
```

### Creation of a PV

The previous yaml file creates a hostPath volume with 1Gi of storage. The volume is mounted in the host at `/host/voldata`. Create the PV with the regular `kubectl apply -f <file-name.yaml>` command.

```bash
$ kubectl apply -f hostdata-pv.yaml
persistentvolume/hostdata-pv created

$ kubectl get pv
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
hostdata-pv    1Gi        RWO            Retain           Available           host                    3m
```

## Persistent volume claims (PVC)

The `PersistentVolumeClaim` objects are namespace-scoped resources that are used by developers to request storage. The `PersistentVolumeClaim` objects are bound to `PersistentVolume` objects that meet the requirements criteria.

`PersistentVolumeClaim` (PVC) requests to consume a storage resource, once bound to a `PersistentVolume`, the PVC can be mounted in pods like any regular volume.

### YAML Definition of a PVC

Create the `PersistentVolumeClaim`: 
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hostdata-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  storageClassName: host
```

### Creation of a PVC

Create the PVC with the regular `kubectl apply -f <file-name.yaml>` command. Verify that the PVC is bound to the PV with the command `kubectl get pvc <pvc-name>`, the column `STATUS` must be `Bound`.

```bash
$ kubectl apply -f hostdata-pvc.yaml
persistentvolumeclaim/hostdata-pvc created

$ kubectl get pvc hostdata-pvc
NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
hostdata-pvc   Bound    hostdata-pv   1Gi        RWO            host           15m
```

### Assign a PVC to a Pod

Modify the Pod template:

Under `spec.containers.volumeMounts`, mount the new volume to the container:

```yaml
  volumeMounts:
  - name: hostdata
    mountPath: /data
```

Under `spec.volumes`, specify the new volume at the Pod:

```yaml
volumes:
- name: hostdata
  persistentVolumeClaim:
    claimName: hostdata-pvc
```

## Dynamic provisioning with Storage Classes

Dynamic provisioning allows storage volumes to be created on-demand. This feature eliminates the need for cluster administrators to pre-provision storage. Instead, it automatically provisions storage when it is requested by users.

A `StorageClass` provides a way for administrators to describe the classes of backend persistent storages. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators.

Each cluster has a default `StorageClass`, which is used when a `PersistentVolumeClaim` doesn't specify a `StorageClassName`. The default `StorageClass` is usually set to provision storage on the cloud provider, but it can be changed to provision storage on-premises.

A Storage Class essentially contains two things:

- **Name**: This is the name that will be used to reference the Storage Class.
- **Provisioner**: This defines the underlying storage technology. For example, provisioner would be `efs.csi.aws.com` for Amazon EFS, `ebs.csi.aws.com` for Amazon EBS, `pd.csi.storage.gke.io` for Google Compute Engine (GCE) Persistent Disk (PD), `ssd.csi.azure.com` for Azure Managed Disk, etc.

The SCI driver must be installed in the cluster before creating the `StorageClass`.

### YAML Definition of a StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
parameters: # specific parameters for the provisioner
  provisioningMode: efs-ap
  fileSystemId: fs-026b7b7b
  directoryPerms: "700"
```

### Creation of a StorageClass

Create the `StorageClass` with the regular `kubectl apply -f <file-name.yaml>` command. Verify that the `StorageClass` is created with the command `kubectl get sc`.

```bash
$ kubectl apply -f ebs-sc.yaml
storageclass.storage.k8s.io/ebs-sc created

$ kubectl get sc
NAME            PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
ebs-sc          ebs.csi.aws.com        Delete          Immediate              false                  3m
```

### PVC with a StorageClass

Create a PVC with the `StorageClass` name:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-pvc
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
  storageClassName: ebs-sc # StorageClass name must match with StorageClass definition name
```

Verify the PVC is created and bound to a PV:

```bash
$ kubectl apply -f ebs-pvc.yaml
persistentvolumeclaim/ebs-pvc created

$ kubectl get pvc ebs-pvc
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
ebs-pvc   Bound    pvc-0b0b0b0b-0b0b-0b0b-0b0b-0b0b0b0b0b0b   100Mi      RWX            ebs-sc         3m
```

Now it is possible to use the PVC in a Pod.

### PVC with a StorageClass and a volume binding mode

The `volumeBindingMode` field in the `StorageClass` object defines when a PV is bound to a PVC. The default value is `Immediate`, which means that a PV is bound as soon as the PVC is created. The other value is `WaitForFirstConsumer`, which means that a PV is bound when the PVC is used by a Pod.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-026b7b7b
  directoryPerms: "700"
volumeBindingMode: WaitForFirstConsumer
```

## Common issues

### PVC does not bound with a PV

The main goal of a PVC is to bind to a PV, but if the PVC does not match a PV, the PVC will be in `Pending` status.

```bash
$ kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
persistentvolume/pv-log   100Mi      RWX            Retain           Available           manual                  8m51s
NAME                                STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/claim-log-1   Pending                                                     23s
```

Some reasons for the PVC to be in `Pending` status are:
- The PV is not available.
- The PV is not in the same namespace as the PVC.
- The PV does not have 
  - enough capacity.
  - required access mode.
  - required storage class.
  - required volume binding mode.
  - required node affinity or selector.

After bounding the PVC to the PV, the status of the PVC will change to `Bound`:

```bash
$ kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
persistentvolume/pv-log   100Mi      RWX            Retain           Bound    default/claim-log-1   manual                  11m

NAME                                STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/claim-log-1   Bound    pv-log   100Mi      RWX            manual         3m
``` 

Once the PV and PVC are bounded, the PVC can be used in a Pod:

```yaml
(...)
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /var/log/nginx
      name: log-volume # must match with .spec.volumes.name

  volumes:
  - name: log-volume # must match with .spec.containers.volumeMounts.name
    persistentVolumeClaim: 
      claimName: claim-log-1 # name of the PVC
```

### Deleting a PVC

If the PVC is deleted, but there is a Pod using it, the PVC will stuck in `Terminating` status until the Pod is deleted:

```bash
$ kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
persistentvolume/pv-log   100Mi      RWX            Retain           Bound    default/claim-log-1   manual                  30m

NAME                                STATUS        VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/claim-log-1   Terminating   pv-log   100Mi      RWX            manual         21m
```

After deleting the Pod, the PVC will be finally deleted and the PV will be available again with the status `Released`:

```bash
$ kubectl delete pod webapp 
pod "webapp" deleted

controlplane ~ ➜  kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                 STORAGECLASS   REASON   AGE
persistentvolume/pv-log   100Mi      RWX            Retain           Released   default/claim-log-1   manual                  32m
```

****
## References

- [K8s - Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
- [K8s - Persistent Volume Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
- [K8s - Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
- [GCP - Persistent volumes and dynamic provisioning](https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes)
- [GCP - Using the Compute Engine persistent disk CSI Driver](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/gce-pd-csi-driver)
- [AWS - Persistent storage for Kubernetes](https://aws.amazon.com/blogs/storage/persistent-storage-for-kubernetes/)
- [AWS - Amazon EFS CSI driver](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html)
- [AWS - Amazon EBS CSI driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)
- [Azure - Persistent volumes in Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/concepts-storage#persistent-volumes)