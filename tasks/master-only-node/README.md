# Run pod on master node
## Task:
Let's setup and run nginx pod, which is run on master node only

## Solution
Let's prepare yaml for our POD first
```
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```
Okay. We can do it in super-easy way just with setting up the nodeName

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  nodeName: minikube
status: {}
```
But that works if you already know the name of the master node. 
Let's do it little bit more flexible using affinity rules and tolerations. 

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
          - matchExpressions:
              - key: "node-role.kubernetes.io/master"
                operator: Exists
  tolerations:
    - key: "node-role.kubernetes.io/master"
      effect: "NoSchedule"
status: {}
```
Or the same with just nodeSelector
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  nodeSelector:
    node-role.kubernetes.io/master: ""
  tolerations:
    - key: "node-role.kubernetes.io/master"
      effect: "NoSchedule"
```

That's it for now =)
