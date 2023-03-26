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
* ETCD version 3 has API version 2 by default
* change API version to 3 with setting env variable ETCDCTL_API=3
* RAFT consensus algorithm
* store every information to run the k8s: nodes, pods, cofig, secret, account etc

### Kube-API server

* primary management component in k8s
* kubectk -> kube-API server
* Or we can use API directly
* Authenticate user
* validate request
* retrieve data
* update ETCD
* scheduler
* kubelet
* 'ps' command shows kube-apiserver is running on the master node with many parameter for kubelet, etcd and certificate and etc.
* 'kubectl get pods -n kube-system' shows kube-apiserver-<master node name> pod is running.
* configuration files: /etc/kubenetes/manifests/kube-apiserver.haml
* service file: /etc/systemd/system/kube-apiserver.service

### kube controller manager

 * manage various managers
 * controller: continuously monitor the state of various components within the system and maintain the state to be desired state
 * node controller checks the status of nodes every 5 seconds
 * 'ps' shows kube-controller-manager process
 * service file to run kube-controller-manager.service /etc/systemd/system/kube-controller-manager.service
 * /etc/kubenetes/manifests/kube-controller-manager.yaml
 
### kube scheduler
 
schedule pod on a node: which pod goes which node
 

## DAY2 | 2023-03-28 | 18-27 |
 
### kubelet
 
 * captain on the ship
 * contact pointer from the master ship
 * report status regularly
 * register a node
 * make the container to pull a image and run a container
 * monitor the status of pod
 * use `ps aux` to see the kubelet process
 
### kube-proxy
 
 * every pod can reach every pod
 * an internal network all pods connect to
 * service with an IP cannot join the internal network that is a viraul component, accessible by all pods
 * kube-proxy is a process that runs on each node
   * look for new service
   * create a rule on each node to forward traffic to those services to the backend pods
   * create an IP table rules on each node to forward traffic
   * forward an IP traffic with the service IP to the service pod
   * deployed as a daemonset - gurantee one pod on each node
 
 ### pod
 
 * a container is encapsulated into a pod
 * a pot is a single instance of an application
 * usually 1:1 between pod and application
 
 ### PODs with YAML
 
 Top level properties
 * apiVersion: API version of kubernetes
   * POD: v1
   * Service: v1
   * ReplicaSet: apps/v1
   * Deployment: apps/v1
 * kind:
   * Pod, ReplocaSet, Deployment
 * metadata: Dictionary format for name, labels...
   * Under labels, you can have any kind of key and value paris as many as you want.
 * spec: specify image
   * conatiners property: a list of (name & image) of multiple containers
 
 `kubectl create -f <name>.yml`
 `kubectl describe pod <pot-name>`
 
## DAY3 | 2023-03-29 | 28-35 |
 
 ### ReplicaSets
 
 * High Availability
 * share load: Load balancing and scaling
 
 Replication Controller: parent of pod
 ```
 apiVersion: v1
 kind: ReplicationController
 metadata:
   name: myapp-rc
   labels:
     app: myapp
     type: front-end
 spec:
   template:  ============> just pod definition file below
     metadata:
     ....
     spec:
       containers:
       - name:
         image:
   replicas: 3
 ```
 
 Replication Set
 ```
 apiVersion: apps/v1
 kind: ReplicationSet
 metadata:
   name: myapp-rs
   labels:
     app: myapp
     type: front-end
 spec:
   template:  ============> just pod definition file below
     metadata:
       name: myapp-pod
       labels:
         type: front-end
     spec:
       containers:
       - name:
         image: ==========> end of pod definition
   replicas: 3
   selector: 
     matchLabels:
       type: front-end
 ```
 
 `kubectl create -f <file>.yml`
 
 `kubectl get replicaset`
 
 `kubectl delete replicaset <name>`
 
 `kubectl replace -f <name>.yml`
 
 ### Deployments
 
 Deployment: a Kubernetes object higher layer than ReplicaSet
 * upgrade the instances seamlessly
 * rolling updates
 * undo changes
 * pause, resume changes
 
 definition file is same except the kind Deployment
 ```
 apiVersion: apps/v1
 kind: Deployment
 metadata:
   name: myapp-deployment
   labels:
     app: myapp
     type: front-end
 spec:
   template:
     metadata:
       name: myapp-pod
       labels:
         app: myapp
         type: front-end
     spec:
       containers:
       - name: nginx-container
         image: nginx
   replicas: 3
   selector:
     matchLabels:
       type: front-end
 ```
 
 `kubectl get all`
 
 
## DAY4 | 2023-03-30 | 36-48 |
## DAY5 | 2023-03-31 | 49-61 |
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
