```helm install --namespace default --name openebs stable/openebs ```

NOTES:
The OpenEBS has been installed. Check its status by running:
$ kubectl get pods -n default

For dynamically creating OpenEBS Volumes, you can either create a new StorageClass or
use one of the default storage classes provided by OpenEBS.

Use `kubectl get sc` to see the list of installed OpenEBS StorageClasses. A sample
PVC spec using `openebs-jiva-default` StorageClass is given below:"

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: demo-vol-claim
spec:
  storageClassName: openebs-jiva-default
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5G
---

For more information, please visit http://docs.openebs.io/.

Please note that, OpenEBS uses iSCSI for connecting applications with the
OpenEBS Volumes and your nodes should have the iSCSI initiator installed.


### Choosing the Storage Engine

Types of Storage Engines
OpenEBS provides two types of storage engines.

Jiva - Jiva is the first storage engine that was released in 0.1 version of OpenEBS and is the most simple to use. It is built in GoLang and uses LongHorn and gotgt stacks inside. Jiva runs entirely in user space and provides standard block storage capabilities such as synchronous replication. Jiva is suitable for smaller capacity workloads in general and not suitable when extensive snapshotting and cloning features are a major need. Read more details of Jiva here
cStor - cStor is the most recently released storage engine, which became available from 0.7 version of OpenEBS. cStor is very robust, provides data consistency and supports enterprise storage features like snapshots and clones very well. It also comes with a robust storage pool feature for comprehensive storage management both in terms of capacity and performance. Together with NDM (Node Disk Manager), cStor provides complete set of persistent storage features for stateful applications on Kubernetes. Read more details of cStor here

Visit [https://docs.openebs.io/docs/next/casengines.html]


### Setting default storage class

```kubectl patch sc <openebs-jiva-default|openebs-cstor-sparse> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'```

