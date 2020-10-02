#### OpenEBS Local PV Hostpath

OpenEBS Dynamic Local PV provisioner can create Kubernetes Local Persistent Volumes using a unique Hostpath (directory) on the node to persist data

## Advantages compared to native Kubernetes hostpath volumes

* OpenEBS Local PV Hostpath allows your applications to access hostpath via StorageClass, PVC, and PV. This provides you the flexibility to change the PV providers without having to redesign * your Application YAML.
* Data protection using the Velero Backup and Restore.
* Protect against hostpath security vulnerabilities by masking the hostpath completely from the application YAML and pod.


## Customize hostpath directory
* By default hostpath volumes will be created under /var/openebs/local directory.
* You can change the BasePath value on sc yaml file.

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-hostpath
  annotations:
    openebs.io/cas-type: local
    cas.openebs.io/config: |
      - name: StorageType
        value: hostpath
      - name: BasePath
        value: /var/local-hostpath
provisioner: openebs.io/local
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

## Notes

* The volumeBindingMode MUST ALWAYS be set to WaitForFirstConsumer. volumeBindingMode: WaitForFirstConsumer instructs Kubernetes to initiate the creation of PV only after Pod using PVC is scheduled to the node.
* If the BasePath does not exist on the node, OpenEBS Dynamic Local PV Provisioner will attempt to create the directory, when the first Local Volume is scheduled on to that node. You MUST ensure that the value provided for BasePath is a valid absolute path.
* As the Local PV storage classes use waitForFirstConsumer, do not use nodeName in the Pod spec to specify node affinity. If nodeName is used in the Pod spec, then PVC will remain in pending state.

## Identifying PV associated with sts
```
kubectl get pods -n openebs -l openebs.io/component-name=openebs-localpv-provisioner
```


## References
* https://docs.openebs.io/docs/next/uglocalpv-hostpath.html
