
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

* find the static pod definition path with --config argument of kubelet -> staticPodPath: field in the config file

# DAY2 2023-04-04 76-93

## Multiple Schedulers

## Configuring Scheduler Profiles

## Monitor Cluster Components

* Node-level metrics: number of node, number of healthy node, performance metrics of CPU, memory, network and disk
* Pod-level metrics: number of pod, cpu and memory consumption of pod
* No full feature monitoring built-in k8s
* kubelet has cAdvisor or Container Advisor in it: that retrieves performance metrics from pods and expose them through the kubelet API
* after installing Metrics Server, you can see
* cpu and memory consumption of each node
```
kubectl top node
```
```
kubectl top pod
```

## Managing Application Logs

* If there is one application in Pod
```
kubectl logs -f <POD name>
```
* If there are multiple app in Pod
```
kubectl logs -f <POD> <container name>
```

## Rolling Updates and Rollbacks

* Rolling update is the default behavior.
* Deployment creates a new replicaset when doing the rolling update: and replace a few pod at once, then destroy old replicaset.
* do rollback
```
kubectl create -f definition.yml
kubectl get deployments
kubectl apply -f definition.yml
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1 ---> not recomment, not same to the definition file
kubectl rollout status deployment/myapp-deloyment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment
```

# DAY3 2023-04-05 94-104

## 96. Commands

* The CMD of the ubuntu image is bash -> exit immediately
```
docker run ubuntu
```

* specify ENTRYPOINT
```
FROM Ubuntu
ENTRYPOINT ["sleep"]
...
$ docker run ubuntu-sleeper 10 --> run "sleep 10"

$ docker run ubuntu-sleeper ------> no operand -> error "missing operand"
```

* set default operand
```
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
...
$ docker run ubuntu-sleeper ------> do "sleep 5" if no operand
$ docker run ubuntu-sleeper 10 ---> do "sleep 10"
```

* override entry-point
```
docker run --entrypoint msleep ubuntu-sleeper 20000
```

## 97. Commands and Arguments

* operand to run the container
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    args: ["10"]
```

* over-ride ENTRYPOINT with "spec.comtainers.command"
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["msleep"]
    args: ["10"]
```

# DAY4 2023-04-06 105-115


# DAY5 2023-04-07 116-128














