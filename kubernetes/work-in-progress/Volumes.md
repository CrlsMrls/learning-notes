# Volumes


- The `spec.volumes` field in the Pod spec defines details about volumes used in the Pod
- The `spec.containers.<container>.volumeMounts` field in the container spec mounts a volume to a specific container at a specific location.
- `hostPath` volumes mount data from a specific location on the host (k8s node). The `hostPath` types can be:
  - `Directory` - Mounts an existing directory on the host,
  - `DirectoryOrCreate` - Mounts a directory on the host, and creates it if it does not already exist,
  - `File` - Mounts an existing single file on the host, or
  - `FileOrCreate` - Mounts a file on the host, and creates it if it does not exist.
- `emptyDir` volumes provide temporary storage that uses the host file system and are removed when Pod is deleted


## Create persistent volumes
`PersistentVolume` (PV) defines an abstract storage resource available for later consumption. It defines the type and amount of storage.

Create the `PersistentVolume`:

```
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

`PersistentVolumeClaim` (PVC) defines a request to consume a storage resource, it binds to a `PersistentVolume` that meets the requirements criteria. The PVC are mounted in pods like any regular volume.


Create the `PersistentVolumeClaim`: 
```
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

```
$ kubectl get pvc hostdata-pvc
NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
hostdata-pvc   Bound    hostdata-pv   1Gi        RWO            host           15m
```

Modify the Pod template:

Under `spec.volumes`, specify the new volume at the Pod:

```
volumes:
- name: hostdata
  persistentVolumeClaim:
    claimName: hostdata-pvc
```

Under `spec.containers.volumeMounts`, mount the new volume to the container:

```
  volumeMounts:
  - name: hostdata
    mountPath: /data
```
