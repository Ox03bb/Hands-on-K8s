<img src="./assets/persistentVolume/pv.drawio.svg" alt="LoadBalancer Service" width="60"/>

# Kubernetes Persistent Volumes (PV)

## Introduction

A **PersistentVolume (PV)** is a cluster resource that represents
storage made available to Kubernetes. Unlike a Pod's ephemeral
filesystem, a PV exists independently of Pods and can outlive them.

Storage flow:

``` text
Physical Storage
        │
        ▼
PersistentVolume (PV)
        │
        ▼
PersistentVolumeClaim (PVC)
        │
        ▼
Pod Volume (volumes:)
        │
        ▼
Container (volumeMounts or volumeDevices)
```


## Why PVs exist

Without persistent storage, all data inside a container is lost when the
container is removed.

A PV provides:

-   Persistent storage
-   Storage abstraction
-   Decoupling of applications from storage hardware
-   Support for many storage backends


# PersistentVolume Anatomy

``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: usb-pv

spec:
  capacity:
    storage: 50Gi

  volumeMode: Filesystem

  accessModes:
    - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  storageClassName: manual

  hostPath:
    path: /mnt/usb-storage
```

## apiVersion

Specifies the Kubernetes API version.

``` yaml
apiVersion: v1
```


## kind

The object type.

``` yaml
kind: PersistentVolume
```


## metadata

``` yaml
metadata:
  name: usb-pv
```

Unique name of the PV.


## spec

Describes the storage.


## capacity

``` yaml
capacity:
  storage: 50Gi
```

Advertises how much storage the PV provides.

Supported units:

-   Mi
-   Gi
-   Ti
-   Pi


## volumeMode

Determines how the storage is exposed.

### Filesystem (default)

``` yaml
volumeMode: Filesystem
```

Container receives a mounted directory.

    /data
    ├── file.txt
    └── images/

Use with `volumeMounts`.

Typical applications:

-   MySQL
-   PostgreSQL
-   MongoDB
-   Nginx
-   Web applications


### Block

``` yaml
volumeMode: Block
```

Container receives a raw block device.

    /dev/xvda

No filesystem exists.

Use with `volumeDevices`.

Example:

``` yaml
containers:
- name: app
  image: ubuntu

  volumeDevices:
  - name: raw
    devicePath: /dev/xvda
```

Typical use cases:

-   Specialized databases
-   VM platforms
-   Storage software


## accessModes

### ReadWriteOnce (RWO)

One node can mount read/write.

Typical:

-   Databases


### ReadOnlyMany (ROX)

Multiple Pods may read.

Typical:

-   Documentation
-   Static content


### ReadWriteMany (RWX)

Multiple Pods may read and write.

Typical:

-   Shared uploads
-   Media libraries

Requires storage supporting RWX (NFS, Ceph, etc.).


### ReadWriteOncePod (RWOP)

Exactly one Pod in the cluster can mount the volume.


## persistentVolumeReclaimPolicy

### Retain

Keep the storage after PVC deletion.

Best for production databases.

### Delete

Delete storage when the PVC is deleted.

Common with cloud disks.

### Recycle

Deprecated.


## storageClassName

Associates the PV with a StorageClass.

Manual PV:

``` yaml
storageClassName: manual
```

Dynamic provisioning examples:

-   gp3
-   standard
-   longhorn
-   nfs-client


## Volume Sources

A PV uses exactly one storage backend.

Examples:

-   hostPath
-   nfs
-   csi
-   awsElasticBlockStore
-   azureDisk
-   gcePersistentDisk

Example:

``` yaml
hostPath:
  path: /mnt/usb-storage
```


# PV vs PVC

  PV                            | PVC
  ------------------------------|------------------------
  Actual storage                | Request for storage
  Created by admin/provisioner  | Created by application
  Exists independently          | Binds to one PV


# Docker Comparison

  Docker           | Kubernetes
  -----------------| ----------------------------------
  Docker Volume    | PersistentVolume
  `docker run -v`  | `volumeMounts`
  `--mount`        | `volumeMounts` / `volumeDevices`
  Named volume     | PV + PVC


# Best Practices

-   Use Filesystem mode unless your application explicitly requires raw
    block devices.
-   Use one PVC per stateful Pod (databases).
-   Share PVCs only when applications intentionally share files.
-   Prefer dynamic provisioning using StorageClasses in production.
-   Use `Retain` for critical data.


# Summary

A PersistentVolume is the storage resource of Kubernetes. It abstracts
physical storage, supports many storage backends, and enables persistent
data across Pod lifecycles. Most applications should use **Filesystem**
mode with **volumeMounts**, while **Block** mode is reserved for
specialized workloads requiring direct access to raw storage devices.
