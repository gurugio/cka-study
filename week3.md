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
```
```
etcdctl snapshot restore -h
```

# DAY2 2023-04-11 144-152



# DAY3 2023-04-12 153-162


# DAY4 2023-04-12 163-173



# DAY5 2023-04-13 174-185
