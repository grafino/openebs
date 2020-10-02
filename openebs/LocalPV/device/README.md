#### OpenEBS Local Persistent Volumes backed by Block Devices

* OpenEBS Dynamic Local PV provisioner can create Kubernetes Local Persistent Volumes using block devices available on the node to persist data, hereafter referred to as OpenEBS Local PV Device volumes.

## Advantages compared to native Kubernetes Local Persistent Volumes.

Dynamic Volume provisioner as opposed to a Static Provisioner.
Better management of the Block Devices used for creating Local PVs by OpenEBS NDM. NDM provides capabilities like discovering Block Device properties, setting up Device Pools/Filters, metrics collection and ability to detect if the Block Devices have moved across nodes.

## Custom storageclass

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-device
  annotations:
    openebs.io/cas-type: local
    cas.openebs.io/config: |
      - name: StorageType
        value: device
    # - name: FSType
    #   value: xfs
    # - name: BlockDeviceTag
    #   value: "busybox-device"
provisioner: openebs.io/local
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

## Notes

* The volumeBindingMode MUST ALWAYS be set to WaitForFirstConsumer. volumeBindingMode: WaitForFirstConsumer instructs Kubernetes to initiate the creation of PV only after Pod using PVC is scheduled to the node.
* The FSType will take effect only if the underlying block device is not formatted. For instance if the block device is formatted with "Ext4", specifying "XFS" in the storage class will not clear Ext4 and format with XFS. If the block devices are already formatted, you can clear the filesystem information using wipefs -f -a <device-path>. After the filesystem has been cleared, NDM pod on the node needs to be restarted to update the Block Device.


## (optional) Block Device Tagging
* You can reserve block devices in the cluster that you would like the OpenEBS Dynamic Local Provisioner to pick up some specific block devices available on the node. You can use the NDM Block Device tagging feature to reserve the devices. For example, if you would like Local SSDs on your cluster for running XXX stateful application. You can tag a few devices in the cluster with a tag named XXX


```$ kubectl get blockdevices -n openebs
NAME                                           NODENAME                           SIZE          CLAIMSTATE   STATUS     AGE
blockdevice-024f756ac7e6443d7a6fd9b113a244c7   k8s-federated-storage-pool-3leof   10737418240   Unclaimed    Active     17h
blockdevice-1edd039fbd32f20da9c566aacf2a619a   k8s-federated-storage-pool-3leof   4293901824    Unclaimed    Inactive   19h
blockdevice-213ddf0c20169d58933de0d6e0d5cb86   k8s-federated-storage-pool-3leof   8588869120    Unclaimed    Inactive   18h
blockdevice-59e2446a8c68b2d00fb5aea120252291   k8s-federated-storage-pool-3leoq   10737418240   Unclaimed    Active     17h
blockdevice-683904e790125ebb099342ca92d9b655   k8s-federated-storage-pool-3leoq   483328        Unclaimed    Active     19h
blockdevice-6bb13f92472a40f70caa39f44dc9aa9c   k8s-federated-storage-pool-3leom   53686025728   Claimed      Active     19h
blockdevice-93b74e5c76521c795679c792cb81d72e   k8s-federated-storage-pool-3leoq   53686025728   Claimed      Active     19h
```

```$ kubectl label bd -n openebs blockdevice-024f756ac7e6443d7a6fd9b113a244c7 openebs.io/block-device-tag=busybox-device```


```$ kubectl get blockdevices -n openebs --show-labels
NAME                                           NODENAME                           SIZE          CLAIMSTATE   STATUS     AGE   LABELS
blockdevice-024f756ac7e6443d7a6fd9b113a244c7   k8s-federated-storage-pool-3leof   10737418240   Unclaimed    Active     17h   kubernetes.io/hostname=k8s-federated-storage-pool-3leof,ndm.io/blockdevice-type=blockdevice,ndm.io/managed=true,openebs.io/block-device-tag=busybox-device```