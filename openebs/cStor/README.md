#### cStor

### Creating cStor Storage Pools

* https://docs.openebs.io/docs/next/ugcstor.html#creating-cStor-storage-pools


## 1 - Get the details of blockdevices attached in the cluster.
```kubectl get blockdevice -n openebs```

* We must attach a volume to every worker node.
* Identify block devices which are unclaimed, unmounted on node and does not contain any filesystem. (Raw device)
* We can see more details by describing the blockdevice.

```
NAME                                           NODENAME                       SIZE          CLAIMSTATE   STATUS     AGE
blockdevice-1deee5ebf258da0aa3de3ace59f8fcb7   federated-storage-pool-3lotj   21473771008   Unclaimed    Active     20h
blockdevice-5b449a1be25eb3a27fd8f71815b8a3a7   federated-storage-pool-3lotr   21473771008   Unclaimed    Active     20h
blockdevice-7c3b0f65f2a0d118c8a61fa1bf829cc0   federated-storage-pool-3lotb   483328        Unclaimed    Active     21h
blockdevice-c9a2f5a62a1342918c55425312fc712b   federated-storage-pool-3lotb   21473771008   Unclaimed    Active     20h
blockdevice-f505a3f715a04e11d488daca15b5c1ac   federated-storage-pool-3lotj   483328        Unclaimed    Active     21h
blockdevice-f9014cf39c86a73160d216e059c11ab3   federated-storage-pool-3lotr   483328        Unclaimed    Active     21h
```

## 2 - Create a StoragePoolClaim configuration.

* Edit cstor-pool-config.yaml and add the block devices.

```
#Use the following YAMLs to create a cStor Storage Pool.
apiVersion: openebs.io/v1alpha1
kind: StoragePoolClaim
metadata:
  name: cstor-disk-pool
  annotations:
    cas.openebs.io/config: |
      - name: PoolResourceRequests
        value: |-
            memory: 2Gi
      - name: PoolResourceLimits
        value: |-
            memory: 4Gi
spec:
  name: cstor-disk-pool
  type: disk
  poolSpec:
    poolType: striped
  blockDevices:
    blockDeviceList:
      - blockdevice-14f3a461bdde1cd820dd3a7819e36c54  #Node federated-storage-pool-3lotr
      - blockdevice-1deee5ebf258da0aa3de3ace59f8fcb7  #Node federated-storage-pool-3lotj
      - blockdevice-96e849167d30baa806a144d134749220  #Node federated-storage-pool-3lotb
---
```

* poolType represents how the data will be written to the disks on a given pool instance on a node. Supported values are striped, mirrored, raidz and raidz2

When the poolType = mirrored , ensure the number of blockDevice CRs selected from each node are an even number. The data is striped across mirrors. For example, if 4x1TB blockDevice are selected on node1, the raw capacity of the pool instance of cstor-disk-pool on node1 is 2TB.

```
blockDevices:
    blockDeviceList:
      - blockdevice-14f3a461bdde1cd820dd3a7819e36c54  #Node federated-storage-pool-3lotr
      - blockdevice-1deee5ebf258da0aa3de3ace59f8fcb7  #Node federated-storage-pool-3lotj
      - blockdevice-96e849167d30baa806a144d134749220  #Node federated-storage-pool-3lotb

```

When the poolType = striped, the number of blockDevice CRs from each node can be in any number. The data is striped across each blockDevice. For example, if 4x1TB blockDevices are selected on node1, the raw capacity of the pool instance of cstor-disk-pool on that node1 is 4TB.

```
blockDevices:
    blockDeviceList:
      - blockdevice-14f3a461bdde1cd820dd3a7819e36c54  #Node federated-storage-pool-3lotr
      - blockdevice-1deee5ebf258da0aa3de3ace59f8fcb7  #Node federated-storage-pool-3lotj
      - blockdevice-5b449a1be25eb3a27fd8f71815b8a3a7  #Node federated-storage-pool-3lotr
      - blockdevice-96e849167d30baa806a144d134749220  #Node federated-storage-pool-3lotb
      - blockdevice-a6615702dd287d47f253d026abfcb235  #Node federated-storage-pool-3lotj
      - blockdevice-c9a2f5a62a1342918c55425312fc712b  #Node federated-storage-pool-3lotb
```


When the poolType = raidz, ensure that the number of blockDevice CRs selected from each node are like 3,5,7 etc. The data is written with single parity. For example, if 3x1TB blockDevice are selected on node1, the raw capacity of the pool instance of cstor-disk-pool on node1 is 2TB. 1 disk will be used as a parity disk.

When the poolType = raidz2, ensure that the number of blockDevice CRs selected from each node are like 6,8,10 etc. The data is written with dual parity. For example, if 6x1TB blockDevice are selected on node1, the raw capacity of the pool instance of cstor-disk-pool on node1 is 4TB. 2 disks will be used for parity.

## 3 - Apply the StoragePoolClaim configuration.
``` 
$ kubectl apply -f cstor-pool-mirrored-config.yaml
```

* Get Storage pool configuration
```
$ kubectl get spc
NAME                       AGE
cstor-disk-pool-mirrored   29s
```

* Get cStor Pool

```
$ kubectl get csp
NAME                            ALLOCATED   FREE    CAPACITY   STATUS    READONLY   TYPE       AGE
cstor-disk-pool-mirrored-4jec   272K        19.9G   19.9G      Healthy   false      mirrored   2m43s
cstor-disk-pool-mirrored-iat3   272K        19.9G   19.9G      Healthy   false      mirrored   2m43s
cstor-disk-pool-mirrored-zjp3   81.5K       19.9G   19.9G      Healthy   false      mirrored   2m43s
```

* We can see there are a storage class per each node running on 3 pods (nodes)
```
$ kubectl get pods -n openebs
NAME                                                              READY   STATUS    RESTARTS   AGE
cstor-disk-pool-mirrored-4jec-85c5fbdbb9-7gvjt                    3/3     Running   0          13m
cstor-disk-pool-mirrored-iat3-556b4d5554-7wxzc                    3/3     Running   0          13m
cstor-disk-pool-mirrored-zjp3-59ff464cdd-p796h                    3/3     Running   0          13m
```

* Get the sc
```
$ kubectl get sc
NAME                         PROVISIONER                                                RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-cstor-sc             openebs.io/provisioner-iscsi                               Delete          Immediate              false                  32s
openebs-device               openebs.io/local                                           Delete          WaitForFirstConsumer   false                  22h
openebs-hostpath             openebs.io/local                                           Delete          WaitForFirstConsumer   false                  22h
openebs-jiva-default         openebs.io/provisioner-iscsi                               Delete          Immediate              false                  22h
openebs-snapshot-promoter    volumesnapshot.external-storage.k8s.io/snapshot-promoter   Delete          Immediate              false                  22h
```

## 4 - Creating cStor Storage Class
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: openebs-scstor-sc
  annotations:
    openebs.io/cas-type: cstor
    cas.openebs.io/config: |
      - name: StoragePoolClaim
        value: "cstor-disk-pool-mirrored"
      - name: ReplicaCount
        value: "3"
provisioner: openebs.io/provisioner-iscsi
```

## 5 - Deploy a StatefulSet
* On the Sts the storageClassName must mention the recently created sc.



### Setting Pool Policies

### Monitoring cStor

helm install grafana grafana/grafana


kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo


helm install prometheus prometheus-community/prometheus --set alertmanager.enabled=false --set server.persistentVolume.storageClass=openebs-cstor-sc
prometheus-server.default.svc.cluster.local




### Deleting or Updating a storage pool using same (or some) disks

* 1 - Terminate the Sts that has OpeEBS sc in use.
* 2 - Terminate the PVC bound between the Sts and PV.
* 3 - Delete the sp configuration to free the blockdevices.
* 4 - Delete the sc because is not available anymore.

### Notes

* Noticed that the container was always in Terminating state after deletion of sts.
* Had to force the deletion of pod and pvc.



## Custom resources

kubectl get cvr -n openebs
kubectl get cstorvolume -n openebs

