
# DAY1 2023-04-03 62-75

 
 ## Node Affinity
 
 * Node Selectors feature is limited: does not provide advanced selector "OR", "NOT".
 * add Node Selector into pod definition file
 ```
 apiVersion: v1
 kind: Pod
 metadata:
   name: myapp-pod
 spec:
   containers:
   - name: data-processor
     image: data-processor
 affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: size
           operator: In
           values:
           - Large
           - Medium ----> Large or Medium
 ```
 
 ```
 - matchExpressions:
   - key : size
     operator: NotIn
     values:
     - Small    ----> except Small node?
 ```
 
 * Node Affinity Types
   * requiredDuringSchedulingIgnoredDuringExecution: pod cannot start if affinity cannot meet
   * preferredDuringSchedulingIgnoredDuringExecution: ignore affinity if there is no matching node
 
 ### Resource Requirements and Limits
 
 * add "spec: resources:" field to set minimum, "spec: limits: " set limits.
 ```
 apiVersion: v1
 kind: Pod
 metadata:
   name: simple-webapp-color
   labels:
     name: simple-webapp-color
 spec:
   containers:
   - name: simple-webapp-color
     image: simple-webapp-color
     ports:
     - containerPort: 8080
     resources:
       requests:
         memory: "1Gi"
         cpu: 1
       limits:
         memory: "2Gi"
         cpu: 2
 ```
 
 * Default limits is 1 vCPU and 512 MiB
 * Cpu consuming than the limit will be throttled.
 * Memory consuming over the limit kills the process.
 
### DaemonSets

* ensure one pod on each node
* new node -> run new pod
* monitoring, log
* kube-proxy, network agent

* similar to replicasets
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```

## Static Pods

* kubelet can work without API server, scheduler and etc in master node.
* kubelet read definition files from specific path -> kubelet check it regularly -> create pods
* change the files -> kubelet changes pods
* remove the files -> kubelet deletes pods
* specify the path with --pod-manifest-path option for kubelet command
* add --config=filename.yaml and staticPodPath value in filename.yaml

* check pod with 'docket ps' on the node
* You cannot modify the static pod with kube API and kubectl.
* ignore by the kube-scheduler

# DAY2 2023-04-04 76-93


# DAY3 2023-04-05 94-104


# DAY4 2023-04-06 105-115


# DAY5 2023-04-07 116-128
