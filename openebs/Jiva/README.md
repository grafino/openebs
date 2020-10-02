### Jiva

* Jiva is a light weight storage engine that is recommended to use for low capacity workloads. 
* OpenEBS provisions a Jiva volume with three replicas on three different nodes. Ensure that there are 3 Nodes in the cluster. The data in each replica is stored in the local container storage of that replica itself. 
* The data is replicated and highly available and is suitable for quick testing of OpenEBS and simple application PoCs. 
* If it is single node cluster, then change the replica count accordingly and apply the modified YAML spec.
* If its a several cluster node but you want to bound to a mount point in a certain node specify the host in the Jiva sp.


## Provision with local attached or cloud storage

* In this mode, local disks on each node has to be formatted and mounted at a directory path. 
* All nodes must have the same mount point.
* Need to create an aditional storage pool.

```
apiVersion: openebs.io/v1alpha1
kind: StoragePool
metadata:
    name: jiva-pool            
    type: hostdir
spec:
    host: k8s-federated-storage-pool-3leom
    path: "/mnt/5g"
```



## Storage Class

## References
* https://docs.openebs.io/docs/next/jivaguide.html