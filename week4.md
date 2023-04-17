# DAY1 2023-04-17 186-198

## 186. Volumes

* Permanent volume stores data permanently.
* Normal volume is deleted when deleting container.
* Permanent volume data remains after deleting container.

```
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume
 volumes:
 - name: data-volume
   hostPath:
     path: /data
     type: Directory

```

## 187. Persistent Volumes

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
  - ReadWriteOnde
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
```

## Persistent Volume Claims

* Admin creates PV
* User creates persistent volume claims
* Kube binds the persistent volumes to claims based on the request and property set on the volume
* 1:1 mappint between PVC and PV.
* And also use labels and selector

* PVC definition
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnde
  resources:
    requests:
      storage: 500Mi 
  persistentVolumeReclaimPolicy: Recycle (scrub storage), Delete, Retain(do not delete data) 
```

```
kubectl get persistentvolumeclaim ==> check status: Pending or Bound
```

```
kubectl delete persistentvolumeclaim myclaim
```

* K8s looks up the PVs and match PVC to one PV

## 194. Storage Class

* Dynamic provisioning of volumes
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
```

* We don't need PV definition anymore because PV is created automatically when the SC is created.

* change PVC to use SC
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnde
  storageClassName: google-storage =====> use SC
  resources:
    requests:
      storage: 500Mi 
```

* When pvc is created, SC will create a PV automatically and bind it.
* There are many provisioner.
* Each provisioner has its own parameters.


# DAY2 2023-04-18 199-209



# DAY3 2023-04-19 210-223


# DAY4 2023-04-20 224-231



# DAY5 2023-04-21 232-241





