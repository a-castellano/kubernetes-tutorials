# Configure Pods and Containers

## Assign Memory Resources to Containers and Pods

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/)

Wait until metrics-server is ready
```bash
$  kubectl get apiservices | grep metrics.k8s.io
v1beta1.metrics.k8s.io                 kube-system/metrics-server   True        2m
```

Create a namespace
```bash
$  kubectl create namespace mem-example
namespace/mem-example created
```

### Specify a memory request and a memory limit

```bash
kubectl create -f pods/resource/memory-request-limit.yaml
pod/memory-demo created
```

Top
```bash
kubectl top pod memory-demo --namespace=mem-example
NAME          CPU(cores)   MEMORY(bytes)
memory-demo   114m         150Mi
```

Delete pod
```bash
kubectl delete pod memory-demo --namespace=mem-example
```

### Exceed a Container’s memory limit

```bash
kubectl create -f pods/resource/memory-request-limit-2.yaml
pod/memory-demo-2 created

kubectl get pod memory-demo-2 --namespace=mem-example
NAME            READY   STATUS    RESTARTS   AGE
memory-demo-2   1/1     Running   2          21s

kubectl get pod memory-demo-2 --namespace=mem-example
NAME            READY   STATUS      RESTARTS   AGE
memory-demo-2   0/1     OOMKilled   2          24s
```

### Specify a memory request that is too big for your Nodes
```bash
kubectl create -f pods/resource/memory-request-limit-3.yaml
pod/memory-demo-3 created

kubectl get pod memory-demo-3 --namespace=mem-example
NAME            READY   STATUS    RESTARTS   AGE
memory-demo-3   0/1     Pending   0          25s
```

Describe will show
```bash
Events:
  Type     Reason            Age               From               Message
  ----     ------            ----              ----               -------
  Warning  FailedScheduling  2s (x4 over 82s)  default-scheduler  0/4 nodes are available: 1 node(s) were unschedulable, 3 Insufficient memory.
```
### Clean up
```bash
kubectl delete pod memory-demo-3 --namespace=mem-example
kubectl delete namespace mem-example
```

## Assign CPU Resources to Containers and Pods

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

### Specify a CPU request and a CPU limit

Create container:
```bash
kubectl create -f pods/resource/cpu-request-limit.yaml
pod/cpu-demo created
```

Check CPU consumption:
```bash
kubectl top pod cpu-demo --namespace=cpu-example
NAME       CPU(cores)   MEMORY(bytes)
cpu-demo   1001m        1Mi
```

Delete pod
```bash
kubectl delete pod cpu-demo --namespace=cpu-example
```

### Specify a CPU request that is too big for your Nodes

Create container:
```bash
kubectl create -f pods/resource/cpu-request-limit-2.yaml
```

Get status:
```bash
kubectl get pod cpu-demo-2 --namespace=cpu-example
NAME         READY   STATUS    RESTARTS   AGE
cpu-demo-2   0/1     Pending   0          29s
```

Describe
```bash
kubectl describe pod cpu-demo-2 --namespace=cpu-example
Name:               cpu-demo-2
Namespace:          cpu-example
Priority:           0
PriorityClassName:  <none>
Node:               <none>
Labels:             <none>
Annotations:        <none>
Status:             Pending
IP:
Containers:
  cpu-demo-ctr-2:
    Image:      vish/stress
    Port:       <none>
    Host Port:  <none>
    Args:
      -cpus
      2
    Limits:
      cpu:  100
    Requests:
      cpu:        100
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-ffr79 (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  default-token-ffr79:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-ffr79
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age                From               Message
  ----     ------            ----               ----               -------
  Warning  FailedScheduling  10s (x4 over 72s)  default-scheduler  0/4 nodes are available: 1 node(s) were unschedulable, 3 Insufficient cpu.
```

kubectl delete namespace cpu-example
```bash
kubectl delete pod cpu-demo-2 --namespace=cpu-example
kubectl delete namespace cpu-example
```

### Configure Quality of Service for Pods

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/)

Create a nampespace
```bash
kubectl create namespace qos-example
```
#### Guaranteed

For a Pod to be given a QoS class of Guaranteed:

* Every Container in the Pod must have a memory limit and a memory request, and they must be the same.
* Every Container in the Pod must have a CPU limit and a CPU request, and they must be the same.

Creates qos-pod
```bash
kubectl create -f pods/qos/qos-pod.yaml
```

Check pod data:
```bash
kubectl get pod qos-demo --namespace=qos-example --output=yaml
```

Delete this pod
```bash
kubectl delete pod qos-demo --namespace=qos-example
```

#### Burstable

A Pod is given a QoS class of Burstable if:

* The Pod does not meet the criteria for QoS class Guaranteed.
* At least one Container in the Pod has a memory or CPU request

Create a pod
```bash
kubectl create -f pods/qos/qos-pod-2.yaml
```

Get pod data
```bash
kubectl get pod qos-demo-2 --namespace=qos-example --output=yaml
```

Delete pod
```bash
kubectl delete pod qos-demo-2 --namespace=qos-example
```

#### BestEffort

For a Pod to be given a QoS class of BestEffort, the Containers in the Pod must not have any memory or CPU limits or requests.

Create a pod
```bash
kubectl create namespace qos-example
namespace/qos-example created
```

Examine the new pod
```bash
kubectl get pod qos-demo-3 --namespace=qos-example --output=yaml | grep qosClass
  qosClass: BestEffort
```

Delete the pod
```bash
kubectl delete pod qos-demo-3
```

#### Create a Pod that has two Containers

Create the pod
```bash
kubectl create -f pods/qos/qos-pod-4.yaml
```

Check that pod qos is Burstable:
```bash
kubectl get pod qos-demo-4 --namespace=qos-example --output=yaml | grep qosClass
  qosClass: Burstable
```

Delete it
```bash
kubectl delete pod qos-demo-4 --namespace=qos-example
```

Clean up
```bash
kubectl delete namespace qos-example
```

## Assign Extended Resources to a Container

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/extended-resource/)

### Assign an extended resource to a Pod

```bash
$ kubectl get pod extended-resource-demo
NAME                     READY   STATUS    RESTARTS   AGE
extended-resource-demo   0/1     Pending   0          3m49s
```

This does not work
```bash
$ kubectl describe pod extended-resource-demo
Name:               extended-resource-demo
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               <none>
Labels:             <none>
Annotations:        <none>
Status:             Pending
IP:
Containers:
  extended-resource-demo-ctr:
    Image:      nginx
    Port:       <none>
    Host Port:  <none>
    Limits:
      example.com/dongle:  3
    Requests:
      example.com/dongle:  3
    Environment:           <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rf5pv (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  default-token-rf5pv:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-rf5pv
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age                    From               Message
  ----     ------            ----                   ----               -------
  Warning  FailedScheduling  3m55s (x2 over 3m55s)  default-scheduler  0/2 nodes are available: 2 Insufficient example.com/dongle.
```

Now it works
```bash
$ kubectl get pod extended-resource-demo
NAME                     READY   STATUS    RESTARTS   AGE
extended-resource-demo   1/1     Running   0          18m
```

Get dongles.
```bash
Containers:
  extended-resource-demo-ctr:
    Container ID:   docker://c4a77012122b43b6949a7999c98a7ed0818181602ff7a981101e6ec937e431a9
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:dd2d0ac3fff2f007d99e033b64854be0941e19a2ad51f174d9240dda20d9f534
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 24 Feb 2019 18:11:43 +0100
    Ready:          True
    Restart Count:  0
    Limits:
      example.com/dongle:  3
    Requests:
      example.com/dongle:  3
```

Attempt to created a second pod
```bash
kubectl create -f pods/resource/extended-resource-pod-2.yaml
pod/extended-resource-demo-2 created
```

There are not enought dongles
```bash
kubectl describe pod extended-resource-demo-2

Events:
  Type     Reason            Age                    From               Message
  ----     ------            ----                   ----               -------
  Warning  FailedScheduling  2m31s (x2 over 2m31s)  default-scheduler  0/2 nodes are available: 2 Insufficient example.com/dongle.
```

Clean all
```bash
kubectl delete pod extended-resource-demo
kubectl delete pod extended-resource-demo-2
```

### Configure a Pod to Use a Volume for Storage

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

```
An emptyDir volume is first created when a Pod is assigned to a Node, and exists as long as that Pod is running on that node.
As the name says, it is initially empty. Containers in the Pod can all read and write the same files in the emptyDir volume,
though that volume can be mounted at the same or different paths in each Container. When a Pod is removed from a node for any reason,
the data in the emptyDir is deleted forever.
```

Configure a volume for a Pod
```bash
kubectl create -f  pods/storage/redis.yaml
```

Enter the pod
```bash
kubectl exec -it redis -- /bin/bash
```

Inside the container place a file
```bash
cd /data/redis/
echo Hello > test-file
```

Install procps and kill redis
```bash
apt-get update
apt-get install procps

ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
redis        1  0.1  0.0  50292  3992 ?        Ssl  19:51   0:00 redis-server *:6379
root        19  0.0  0.0  18144  3204 pts/0    Ss   19:52   0:00 /bin/bash
root       280  0.0  0.0  36640  2800 pts/0    R+   20:00   0:00 ps -aux
root@redis:/data/redis# kill -9 1
```

Container restarts
```bash
$ kubectl get pod redis --watch
NAME    READY   STATUS    RESTARTS   AGE
redis   1/1     Running   0          92m
redis   0/1   Completed   0     92m
redis   1/1   Running   1     93m
redis   0/1   Completed   1     103m
redis   1/1   Running   2     103m
```

Get back to the new container
```bash
kubectl exec -it redis -- /bin/cat /data/redis/test-file
Hello
```

Destroy the pod
```bash
kubectl delete pod redis
```

### Configure a Pod to Use a PersistentVolume for Storage

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

### Configure a Pod to Use a Projected Volume for Storage

[Link](Configure a Pod to Use a Projected Volume for Storage)

Create the Secrets:
```bash
echo -n "admin" > ./username.txt
echo -n "1f2d1e2e67df" > ./password.txt
kubectl create secret generic user --from-file=./username.txt
kubectl create secret generic pass --from-file=./password.txt
```

Create pod
```bash
kubectl create -f pods/storage/projected.yaml
```

Check secrets
```bash
kubectl exec -it test-projected-volume -- ls -l /projected-volume/
total 0
lrwxrwxrwx    1 root     root            19 Feb 24 21:02 password.txt -> ..data/password.txt
lrwxrwxrwx    1 root     root            19 Feb 24 21:02 username.txt -> ..data/username.txt
```

```bash
kubectl exec -it test-projected-volume -- cat /projected-volume/password.txt
1f2d1e2e67df
kubectl exec -it test-projected-volume -- cat /projected-volume/username.txt
admin
```

```bash
```

```bash
```

```bash
```

```bash
```

```bash
```

```bash
```
