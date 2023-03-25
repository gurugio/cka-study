# Week-1

## DAY1 | 2023-03-27 | 1-17, 273(실습 준비) |

### High level architecture

Control Plane (Master node): manager, plan, scheduler, monitor
* kube-apiserver: API server exposes the Kubernetes API, front end of the k8s control plane
  * main implementation of k8s api server is kube-apiserver
  * manage all component, orchestrate all operations within the clustor
* etcd: key-value store, backing store for all cluster data
* kube-scheduler: select a node for them to run on
* kube-controller-manager: run controller processes
  * node controller
  * job controller
  * endpointslice controller
  * serviceaccount controller
  * replication controller
* cloud-controller-manager: embeds cloud-specific control logic

Container runtime engine: Docker, Containerd, etc

Workder Node Components: Host applications as containers
* kubelet: captain, listen to kube-apiserver, agent, run on each node, make sure that containers are running in a Pod
* kube-proxy: run on each node, communication between nodes, maintains network rules on nodes, allow network communication to pods, use packet filtering layer?

### ETCD

key-value store
* traditional DB has row and column -> Adding a column affects all data -> many empty cells
* key-value store
  * store information in the form of documents or pages
  * For example, each individual gets a document and all info of an individual is store within a file
  * change one file does not affect the others
  * different file has different key-value
  * add info without having to update others
  * extract data to Json, Yaml format

ETCD version 3 has API version 2 by default
change API version to 3 with setting env variable ETCDCTL_API=3

RAFT consensus algorithm

store every information to run the k8s: nodes, pods, cofig, secret, account etc

### Kube-API server

primary management component in k8s

kubectk -> kube-API server
Or we can use API directly

Authenticate user
validate request
retrieve data
update ETCD
scheduler
kubelet

'ps' command shows kube-apiserver is running on the master node with many parameter for kubelet, etcd and certificate and etc.

'kubectl get pods -n kube-system' shows kube-apiserver-<master node name> pod is running.
 
configuration files: /etc/kubenetes/manifests/kube-apiserver.haml

service file: /etc/systemd/system/kube-apiserver.service





## DAY2 | 2023-03-28 | 18-27 |
## DAY3 | 2023-03-29 | 28-35 |
## DAY4 | 2023-03-30 | 36-48 |
## DAY5 | 2023-03-31 | 49-61 |
