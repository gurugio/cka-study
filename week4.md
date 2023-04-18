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

## Network Namespaces

* create a network namespaces

```
ip netns add red
ip netns add blue
ip netns => list network namespaces
```

* list interface in the red ns
```
ip netns exec red ip link
ip -n red link
ip -n red arp
ip -n red route
```

* setup network
```
ip link add veth-red type veth peer name veth-blue => link between veth
ip link set veth-red netns red => veth <-> network namespace
ip link add veh-blud netns blue

ip -n red addr add 192.168.15.1 dev veth-red ==> set ip
ip -n blue addr add 19.168.15.2 dev veth-blue

ip -n red link set veth-red up
ip -n blue link set veth-red up

ip -n red ping 192.168.15.2


ip -n red arp
ip -n blue arp
```

* Create a virtual switch with Linux Bridge, Open vSwitch 
* How to create a virtual switch with Linux Bridge
```
ip link add v-net-0 type bridge ==> create another interface with bridge type
ip link ===> list v-net-0 along with other interfaces
ip link set dev v-net-0 up

; connect all namespaces to the bridge network
ip link add veth-red type veth peer name veth-red-br ; veth-red <-> veth-red-br
ip link add veth-blue type veth per name veth-blue-br ; veth-blue <-> veth-blue-br
ip link set veth-red netns red ; use veth-red for red namespace
ip link set veth-red-br master v-net-0 ; veth-red-br <-> v-net-0 virtual switch
ip -n red addr add 192.168.15.1 dev veth-red
ip -n red link set veth-red up
```

* set an IP to the virtual switch
```
ip addr add 192.168.15.5/24 dev v-net-0
ping 192.168.15.5 ; connect host to the virtual switch
```

* enable external network
```
ip netns exec blue ping 192.168.1.3 ; does not work
ip netns exec blue route ; only 192.168.15.0
ip -n blue ip route add 192.168.1.0/24 via 192.168.15.5 ; add gateway, gateway is v-net-0 of the host
iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE ; use NAT
```


## Docker Networking

* host network: `docker run --network host nginx` same network with host
* Two container cannot listen to the same port

* bridge network: internal private network which Docker host and containers attach to `docker run nginx`
* default address is 172.17.0.0
* Each container has its own internal private network address 

```
docker network ls
...
..bridge...
```

* Whenever docker create new container, it creates a network namespace, creates a pair of interfaces (one end to container, another end to bridge)
* Then the all containers can communicate each other.
* Still it needs to map port of container to external port with NAT: HOST's ip:port -> container's IP:port
* Now external use can access the container with host's IP and external port.

## Container networking interface CNI

* common interface to create network


# DAY3 2023-04-19 210-223

## Pod Networking

* k8s networking model
* Every POD should have an IP address
* Every POD should be able to communicate with every other POD in the same node
* Every POD should be able to communicate with every other POD on other nodes without NAT.


* Each node has its own IP
* NODE1: 192.168.1.11
* NODE2: 192.168.1.12
* NODE3: 192.168.1.13

* Create a bridge network on each node
```
node1: ip link add v-net-0 type bridge; ip link set dev v-net-0 up
node2: ip link add v-net-0 type bridge; ip link set dev v-net-0 up
node3: ip link add v-net-0 type bridge; ip link set dev v-net-0 up
```

* assing IP to bridge: each brige network has its own subnet
```
node1: ip addr add 10.244.1.1/24 dev v-net-0
node2: ip addr add 10.244.2.1/24 dev v-net-0
node3: ip addr add 10.244.3.1/24 dev v-net-0
```

* And do many steps: create virtual interfaces between nodeX and container, add route table and etc
* CNI defines the steps to establish network
* kubelet runs a script passed as parameter whenever it creates a container.
* The script implements CNI interfaces.

## CNI in kubernetes

* view kubelet options `ps aux | grep kubelet` -> --cni-bin-dir, --cni-conf-dir, --network-plugin
* /opt/cni/bin: executables of plugins
* /etc/cni/net.d: configuration, kube checks which plugin is used

## CNI weave

* deploy an agent or service on each node
* They comminucate and exchange information regarding the nodes, pod and interfaces
* Weave creates its own bridge on the nodes and namespaces, and allocate IPs to each network.
* Weave intercepts packet to another nodes and handle send/receive for itself.
* install weave
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base 64 | tr -d '\n')" 
```
* create a daemonset to deploy it on all nodes
* `kubectl get pods -n kube-system` -> see weave-net-XXXX on each nodes
* `kubectl logs weave-net-XXXX weave -n kube-system` -> see log

# DAY4 2023-04-20 224-231



# DAY5 2023-04-21 232-241






