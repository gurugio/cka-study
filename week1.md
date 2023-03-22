# Week-1

## DAY1 | 2023-03-27 | 1-17, 273(실습 준비) |

Control Plane
* kube-apiserver: API server exposes the Kubernetes API, front end of the k8s control plane
  * main implementation of k8s api server is kube-apiserver
* etcd: key-value store, backing store for all cluster data
* kube-scheduler: select a node for them to run on
* kube-controller-manager: run controller processes
  * node controller
  * job controller
  * endpointslice controller
  * serviceaccount controller
* cloud-controller-manager: embeds cloud-specific control logic

Node Components
* kubelet: agent, run on each node, make sure that containers are running in a Pod
* kube-proxy: run on each node, maintains network rules on nodes, allow network communication to pods, use packet filtering layer?
* container runtimerun containers, containerd/cri-o and etc

## DAY2 | 2023-03-28 | 18-27 |
## DAY3 | 2023-03-29 | 28-35 |
## DAY4 | 2023-03-30 | 36-48 |
## DAY5 | 2023-03-31 | 49-61 |
