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

## IP address Management - Weave

* IPAM in the CNI plugin manages IP addresses
* /etc/cni/net.d/net-script.conf defines type of plugin, subnet, route


## Service Networking

* A service is accessible from all pods of any node
* Service is not bound to a specific node but hosted across a cluster.
* ClusterIP: provide an IP for service accessible across a cluster => internal access
* NodePort: provide an IP and port for external access

* kubelet create a pod and invokes the CNI plugin to configure networking
* When a new service is created, it is assigned an IP address. kube-proxy gets that IP address and creates forwading rules on each node.
* Service does not exist actually. There is nothing listening no the IP of the service. There is no process.
* A traffic to the service IP:port will be forwared to the IP:port of pod by kube-proxy.
* kube-proxy manages a rule for ip:port forwarding.

`kubectl get service` -> shows IP and port of ClusterIP or NodePort

`kube-apiserver --service-cluster-ip-range ipNet` -> set a IP range for service 

`ps aux | grep kube-apiserver` -> see service-cluster-ip-range


```
iptables -L -t nat | grep db-service
KUBE-SVC-XXXX <ip> <port> <= source ip
DNAT tcp to <ip:port> <= target ip 
```

`cat /var/log/kube-proxy.log` see what proxy it uses: add new service <service-name:port> to <IP:port>


## DNS in kubernetes

* Each node has its own node name and IP
* External DNS server has the node names and IPs

* DNS resolution within the cluster: pod, services and etc
* k8s has default DNS server
* Whenever a service is created, DNS service of k8s creates a record for the service.
* In the same namespace, it needs only service name `http://web-service`
* From another namespace, it appends namespace after the service name `http://web-service.apps`
* sub-domain of service is svc => `http://web-service.apps.svc`
* root domain cluster.local => `http://web-service.apps.svc`
* `http://<IP-of-server>.apps.svc.cluster.local` is same


# DAY4 2023-04-20 224-231

## CoreDNS in Kubernetes

* /etc/resolve.conf : save dns server ipaddress
* k8s deploys a DNS server in the cluster: Kube DNS (old) CoreDNS( since 1.2)
* conf file location: /etc/coredns/Corefile
* That conf file is managed by a configmap: coredns `kubectl get configmap -n kube-system`
* When CoreNDS is deployed, k8s creates a servive "kube-dns"
* When a pod is created, kubelet sets DNS to IP address of kube-dns in /etc/resolv.conf.
```
cat /var/lib/kubelet/config.yaml
...
clusterDNS:
- A.B.C.D =======> IP of kube-dns service
clusterDomain: cluster.local
```
* Then pod can access other pods with name: host command shows full domain name and IP address
```
# host web-service
web-service.default.svc.cluster.local has address A.B.C.D
```

* host command with pod works with full name
```
# host 10-244-2-6
not found
# host 10-244-2-6.default.pod.cluster.local
10-244-2-6.default.pod.cluster.local has address 10.244.2.6
```

## Ingress

* k8s service of deployment, wear-service, exports a high number of port, 30080, for the external user.
* A proxy server or load balancer forwards 30080 port to 80 port and then the external user can access the service via 80 port.
* Create new service (or LoadBalancer), video-service, exports 38202 port.
* Then a new proxy or load-balancer to access two services.
* Total there needs three proxy/load-balancer.

* All of the proxy and load-balander could be replaced with k8s Ingress.
* Ingress makes user access applications with single externally accessible url.
* Ingress also route traffic to different services based on the URL path.
* Ingress supports SSL as well.
* Ingress is a virual layer-7 load balancer builtin.


* Ingress controller: deploying nginx eg. not deployed by default. any layer-7 load balancer.
* Ingress resources: definition files

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
      args:
        - /nginx-ingress-controller
        - --configmap=$(POD_NAMESPACE)/nginx-configuration
      env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
      ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
```

* service to be accessed from external user
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```

* `kubectl create -f ingress-wear.yaml`
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:
    serviceName: wear-service
    servicePort: 80
```

`kubectl get ingress` shows ingress objects.

* Ingress routes traffic based on url
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service ---> wear-service ingress is created above
          servicePort: 80
      - path: /watch
        backend:
          serviceName: watch-service
          servicePort: 80
```

`kubectl describe ingress ingress-wear-watch`: shows which path is mapped to which backend
```
Default backend: default-http-backend:80 ---> user access a patch not listed below
...
/wear wear-service:80
/watch watch-service:80
...

```

* set host name instead path
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - host: wear.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: wear-service ---> wear-service ingress is created above
          servicePort: 80
  - host: watch.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: watch-service
          servicePort: 80
```

# DAY5 2023-04-21 232-241




















