# Setup ghost blog
Yet another challenge which might be useful.
Let's run simple blog engine ghost with mysql persistance.

## Setup pv
We will use mysql as our storage and that will be good to configure Persistent Volume for that. As we are doing that for minikube, that's totally fine to have it created as HostPath.
We will use snippet like that for doing creation:
```

apiVersion: v1
kind: PersistentVolume
metadata:
  name: db-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"

```
[Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

Next step will be creating PVC.
Snippet like that will be used for that
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-claim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Gi
  storageClassName: manual
```

Done! Now it seems like we need to create secrets. Not a big deal. Will use the following command to create secret 
```
kubectl create secret generic db-credentials --from-literal=mysql-password=top-secret-password --from-literal=mysql-username=root
```

It's time to create a pod with my-sql which gonna use created secret. That's kind of easy, but actually that would be nice if we have some placeholers for env variables:
```
kubectl run mysql-db --image=mysql:latest --env=MYSQL_ROOT_PASSWORD=password --env=MYSQL_USER=usr --dry-run=client -o yaml > mysql-pod.yaml
```

Next step will be update created yaml with environment variables from secrets and volume from pvc
```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: mysql-db
  name: mysql-db
spec:
  containers:
  - env:
    - name: MYSQL_ROOT_PASSWORD
      valueFrom:
         secretKeyRef:
           name: db-credentials
           key: mysql-password
    - name: MYSQL_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: mysql-username
    image: mysql:5.6
    name: mysql-db
    resources: {}
    volumeMounts:
       - mountPath: /var/lib/mysql
         name: mysql-pv
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:
    - name: mysql-pv
      persistentVolumeClaim:
         claimName: db-claim
status: {}
```


Now let's create our ghost blog deployment. Well, again, we can start with creating template using 
```
kubectl create deployment ghost-blog --image=ghost:alpine --dry-run=client -o yaml > ghost-deployment.yaml
```
And then modify it in order to contain all required environment variables

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: ghost-blog
  name: ghost-blog
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ghost-blog
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: ghost-blog
    spec:
      containers:
      - image: ghost:alpine
        name: ghost
        env:
         - name: database__client
           value: mysql
         - name: database__connection__user
           valueFrom:
              secretKeyRef:
                 name: db-credentials
                 key: mysql-username
         - name: database__connection__password
           valueFrom:
              secretKeyRef:
                 name: db-credentials
                 key: mysql-password
         - name: database__connection__host
           value: mysql-service
         - name: database__connection__database
           value: ghost
        resources: {}
status: {}

```

let's expose service for created blog via command

```
kubectl expose deployment ghost-blog --name=ghost-blog-service  --type=NodePort --port=2368 
```

Now let's do our challenge a bit more interesting.

## Network policies

As for now, our cluster should contain something like following resources
```
NAME                              READY   STATUS    RESTARTS   AGE
pod/ghost-blog-7f6b68f765-5r5bg   1/1     Running   0          31m
pod/mysql-db                      1/1     Running   0          32m

NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/ghost-blog-service   NodePort    10.106.201.250   <none>        2368:32688/TCP   98m
service/kubernetes           ClusterIP   10.96.0.1        <none>        443/TCP          22d
service/mysql-service        ClusterIP   10.105.95.3      <none>        3306/TCP         134m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ghost-blog   1/1     1            1           106m

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/ghost-blog-7f6b68f765   1         1         1       106m
```

