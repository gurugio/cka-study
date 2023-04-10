# DAY1 2023-04-10 129-143

## 130. Backups and restore methods

* backing up resource configuration
* query kube API server -> save all resource configuration
* Or backup ETCD cluster -> backup ETCD and make a snashot with etcdctl

```
export ETCDCTL_API=3 -----> v2 does not have snapshot command
etcdctl snapshot save -h
--cacert
--cert
--endpoints=[127.0.0.1:2345]
--key
# etcdctl snapshot save /opt/snapshot-pre-boot.db --cacert="/etc/kubernetes/pki/etcd/ca.crt" --cert="/etc/kubernetes/pki/etcd/server.crt" --key="/etc/kubernetes/pki/etcd/server.key" --endpoints=127.0.0.1:2379
Snapshot saved at /opt/snapshot-pre-boot.db
```
```
etcdctl snapshot restore --data-dir /var/lib/etcd-from-backup /opt/snapshot-pre-boot.db

volumes: -hostPath: path: -> change /var/lib/etcd-from-backup -> etcd will be restarted
```

## Kubernetes Security Primitives

* Kube API server is the first line for security
* Who can access / What can they do

* who can access -> authentication
* what can they do - RBAC authorization, ABAC, node, webhook...
* All communications between various modules in master/workder nodes are encrypted with TLS.
* network policy - controll access between Pods

## Authentication



# DAY2 2023-04-11 144-152



# DAY3 2023-04-12 153-162


# DAY4 2023-04-12 163-173



# DAY5 2023-04-13 174-185
