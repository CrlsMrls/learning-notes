# Persistent Volumes and Claims

The `PersistentVolume` (PV) and `PersistentVolumeClaim` (PVC) objects provide a way to use persistent storage with Kubernetes. Developers do not create PV directly, instead they create PVC objects that request storage resources. The PV are bound to PVC that meet the requirements criteria. 


## Persistent volumes

The `PersistentVolume` (PV) objects are cluster-wide resources like a node is a cluster resource. PV are cluster-wide resources that are provisioned by an administrator or dynamically provisioned using [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/).

Each cluster has a default `StorageClass`, which is used to provision storage for PVC that do not specify a StorageClass. The default `StorageClass` is usually set to provision storage on the cloud provider, but it can be changed to provision storage on-premises.

## Access mode

The PersistentVolume resource has a `spec.accessModes` field that specifies what modes the volume can be mounted with. This allows the cluster administrator to control how the volume can be used. These are the available modes:
- `ReadWriteOnce` or `RWO` - The volume can be mounted as read-write by a single node, this is the only mode that can be used with `hostPath` volumes.
- `ReadOnlyMany` or `ROX` - The volume can be mounted read-only by many nodes and many volume types, but is rarely used.
- `ReadWriteMany` or `RWX` - The volume can be mounted as read-write by many nodes. It is only supported by a few volume types, such as NFS and GlusterFS.
- `ReadWriteOncePod` or `RWOPod` - The volume can be mounted as read-write by a single pod. It is only supported by a few volume types, such as `hostPath` and `local`.



## Create persistent volumes
`PersistentVolume` (PV) defines an abstract storage resource available for later consumption. It defines the type and amount of storage.

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

The previous yaml file creates a hostPath volume with 1Gi of storage. The volume is mounted in the host at `/host/voldata` and it is available for pods to use it at `/data`.

## Persistent volume claims

The `PersistentVolumeClaim` objects are namespace-scoped resources that are used by developers to request storage. The `PersistentVolumeClaim` objects are bound to `PersistentVolume` objects that meet the requirements criteria.


`PersistentVolumeClaim` (PVC) defines a request to consume a storage resource, it binds to a `PersistentVolume` that meets the requirements criteria. The PVC are mounted in pods like any regular volume.


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

Verify that the PersistentVolumeClaim is bound to the PersistentVolume:

```bash
$ kubectl get pvc hostdata-pvc
NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
hostdata-pvc   Bound    hostdata-pv   1Gi        RWO            host           15m
```

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

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany 
  hostPath:
    path: "/pv/log"
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim-log-1 
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany 
  resources:
    requests:
      storage: 50Mi
```

If the PVC is not bounded to a PV, the status of the PVC will be `Pending`:

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
  - required volume mode.
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
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
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

If the PVC is deleted, but there is a Pod using it, the PVC will stuck in `Terminating` status until the Pod is deleted:

```bash
$ kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
persistentvolume/pv-log   100Mi      RWX            Retain           Bound    default/claim-log-1   manual                  30m

NAME                                STATUS        VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/claim-log-1   Terminating   pv-log   100Mi      RWX            manual         21m
```

After deleting the Pod, the PVC will be deleted and the PV will be available again with the status `Released`:

```bash
$ kubectl delete pod webapp 
pod "webapp" deleted

controlplane ~ ➜  kubectl get pv,pvc
NAME                      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                 STORAGECLASS   REASON   AGE
persistentvolume/pv-log   100Mi      RWX            Retain           Released   default/claim-log-1   manual                  32m
``````
