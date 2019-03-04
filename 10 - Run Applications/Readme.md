# Run Applications

## Define a Command and Arguments for a Container

[Link](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/)

Create an Nginx deployment:
```bash
k apply -f application/deployment.yaml
```

Check deployment:
```bash
k describe deployment nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Sun, 03 Mar 2019 13:50:39 +0100
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"nginx-deployment","namespace":"default"},"spec":{"replica...
Selector:               app=nginx
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.7.9
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-76bf4969df (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  91s   deployment-controller  Scaled up replica set nginx-deployment-76bf4969df to 2
```

Get pods
```bash
kubectl get pods -l app=nginx

NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-76bf4969df-bb2wb   1/1     Running   0          3m25s
nginx-deployment-76bf4969df-ptw9c   1/1     Running   0          3m25s
```

Update Nginx version
```bash
k apply -f application/deployment-update.yaml
```

Pods have changed:
```bash
kubectl get pods -l app=nginx
```

Scaling the application by increasing the replica count
```bash
k apply -f application/deployment-scale.yaml
```

Now there are 4 pods:
```bash
kubectl get pods -l app=nginx
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5896fbb489-dbqp2   1/1     Running   0          31s
nginx-deployment-5896fbb489-pjxmc   1/1     Running   0          6m51s
nginx-deployment-5896fbb489-qns5b   1/1     Running   0          31s
nginx-deployment-5896fbb489-xqqc5   1/1     Running   0          6m51s
```

Delete deployments:
```bash
k delete deployment nginx-deployment
```

## Run a Single-Instance Stateful Application

[Link](https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/)

Create presistent volume
```bash
k create  -f  application/mysql/mysql-pv.yaml

persistentvolume/mysql-pv-volume created
persistentvolumeclaim/mysql-pv-claim created
```

Create deployment
```bash
k create  -f  application/mysql/mysql-deployment.yaml

service/mysql created
deployment.apps/mysql created
```

Show pods:
```bash
k get pods -l app=mysql

NAME                     READY   STATUS    RESTARTS   AGE
mysql-799956477c-5mzld   1/1     Running   0          8m48s
```

Accessing the MySQL instance
```bash
 kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword
If you don't see a command prompt, try pressing enter.

mysql>
```

## Run a Replicated Stateful Application

[Link](https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/)

Deploy configmap
```bash
k create -f application/mysql/mysql-configmap.yaml
```

Create mysql services
```bash
k create -f  application/mysql/mysql-services.yaml
```

Check pods:
```bash
kubectl get pods -l app=mysql --watch
NAME      READY   STATUS    RESTARTS   AGE
mysql-0   2/2     Running   0          2m43s
mysql-1   2/2     Running   0          92s
mysql-2   2/2     Running   0          57s
```

Create database:
```bash
k run mysql-client --image=mysql:5.7 -i --rm --restart=Never --\
>   mysql -h mysql-0.mysql <<EOF
> CREATE DATABASE test;
> CREATE TABLE test.messages (message VARCHAR(250));
> INSERT INTO test.messages VALUES ('hello');
> EOF
If you don't see a command prompt, try pressing enter.
pod "mysql-client" deleted
```

Check replication:
```bash
k run mysql-client --image=mysql:5.7 -i -t --rm --restart=Never --\
>   mysql -h mysql-read -e "SELECT * FROM test.messages"
+---------+
| message |
+---------+
| hello   |
+---------+
pod "mysql-client" deleted
```

```bash
kubectl run mysql-client-loop --image=mysql:5.7 -i -t --rm --restart=Never --\
>   bash -ic "while sleep 1; do mysql -h mysql-read -e 'SELECT @@server_id,NOW()'; done"
If you don't see a command prompt, try pressing enter.
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         102 | 2019-03-03 19:24:44 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         100 | 2019-03-03 19:24:45 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         101 | 2019-03-03 19:24:46 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         102 | 2019-03-03 19:24:47 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         100 | 2019-03-03 19:24:48 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         101 | 2019-03-03 19:24:49 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         102 | 2019-03-03 19:24:50 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         100 | 2019-03-03 19:24:51 |
+-------------+---------------------+
```

### Simulating Pod and Node downtime

Remove mysql-2
```bash
k exec mysql-2 -c mysql -- mv /usr/bin/mysql /usr/bin/mysql.off
```

Check that mysql-2 unavailable
```bash
kubectl run mysql-client-loop --image=mysql:5.7 -i -t --rm --restart=Never --  bash -ic "while sleep 1; do mysql -h mysql-read -e 'SELECT @@server_id,NOW()'; done"
If you don't see a command prompt, try pressing enter.
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         100 | 2019-03-03 20:02:59 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         101 | 2019-03-03 20:03:00 |
+-------------+---------------------+
+-------------+---------------------+
| @@server_id | NOW()               |
+-------------+---------------------+
|         100 | 2019-03-03 20:03:01 |
+-------------+---------------------+
```

Restore mysql-2
```bash
k exec mysql-2 -c mysql -- mv /usr/bin/mysql.off /usr/bin/mysql
```

Check
```bash
kubectl get pods -l app=mysql --watch
NAME      READY   STATUS    RESTARTS   AGE
mysql-0   2/2     Running   0          59m
mysql-1   2/2     Running   0          58m
mysql-2   2/2     Running   0          57m
```

Scale replicas
```bash
k scale statefulset mysql  --replicas=5
```

Delete statefulset:
```bash
kubectl delete statefulset mysql
```

Delete all
```bash
kubectl delete configmap,service,pvc -l app=mysql
configmap "mysql" deleted
service "mysql" deleted
service "mysql-read" deleted
persistentvolumeclaim "data-mysql-0" deleted
persistentvolumeclaim "data-mysql-1" deleted
persistentvolumeclaim "data-mysql-2" deleted
persistentvolumeclaim "data-mysql-3" deleted
persistentvolumeclaim "data-mysql-4" deleted
```

## Update API Objects in Place Using kubectl patch

[Link](https://kubernetes.io/docs/tasks/run-application/update-api-object-kubectl-patch/)

Use a strategic merge patch to update a Deployment
```bash
k create -f application/deployment-patch.yaml
```

Get pods:
```bash
k get pods -l app=nginx
NAME                          READY   STATUS    RESTARTS   AGE
patch-demo-5b5c885dc9-md85b   1/1     Running   0          91s
patch-demo-5b5c885dc9-vkqkw   1/1     Running   0          91s
```

Patch pods adding a new container:
```bash
k patch deployment patch-demo --patch "$(cat application/patch-file-containers.yaml)"
```

Old pods have been terminated
```bash
k get pods -l app=nginx

NAME                          READY   STATUS    RESTARTS   AGE
patch-demo-69c77dd57d-v4bh2   2/2     Running   0          84s
patch-demo-69c77dd57d-x99rx   2/2     Running   0          98s
```

Patch deployment again:
```bash
k patch deployment patch-demo --patch "$(cat application/patch-file-tolerations.yaml)"
```

Tolerations has been replaced:
```yaml
      tolerations:
      - effect: NoSchedule
        key: dedicated
        value: test-team
```

by

```yaml
      tolerations:
      - effect: NoSchedule
        key: disktype
        value: ssd
```

Path using json merge stategy:
```bash
k patch deployment patch-demo --type merge --patch "$(cat application/patch-file-2.yaml)"
```

Our containers have changed:
```yaml
    spec:
      containers:
      - image: gcr.io/google-samples/node-hello:1.0
        imagePullPolicy: IfNotPresent
        name: patch-demo-ctr-3
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
```

Delete deplyment:
```bash
k delete  deployment patch-demo
```

## Horizontal Pod Autoscaler Walkthrough

[Link](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

Run & expose php-apache server
```bash
k run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m --expose --port=80

kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
service/php-apache created
deployment.apps/php-apache created
```

Set autoscale policy:
```bash
k autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

Get hpa
```bash
kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          81s
```

Increase load:
```bash
k run -i --tty load-generator --image=busybox /bin/sh

while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done
```

hpa works!
```bash
kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   57%/50%   1         10        2          4m50s
```

Get pods
```bash
php-apache-84cc7f889b-fgx9w               1/1     Running   0          72s
php-apache-84cc7f889b-gnshr               1/1     Running   0          8m56s
php-apache-84cc7f889b-htvlz               1/1     Running   0          11s
php-apache-84cc7f889b-rwmnq               1/1     Running   0          12s
```

Stop load killing load-generator and wait:

```bash
kubectl get hpa
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          14m
```

Using file to create HPA
```bash
k create -f application/hpa/php-apache.yaml
```
