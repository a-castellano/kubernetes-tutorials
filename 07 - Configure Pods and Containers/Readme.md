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

## Configure Quality of Service for Pods

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/)

Create a nampespace
```bash
kubectl create namespace qos-example
```
### Guaranteed

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

### Burstable

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

### BestEffort

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

### Create a Pod that has two Containers

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
kubectl get pod extended-resource-demo
NAME                     READY   STATUS    RESTARTS   AGE
extended-resource-demo   0/1     Pending   0          3m49s
```

This does not work
```bash
kubectl describe pod extended-resource-demo
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
kubectl get pod extended-resource-demo
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

## Configure a Pod to Use a Volume for Storage

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
kubectl get pod redis --watch
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

## Configure a Pod to Use a PersistentVolume for Storage

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

Inside your node:
```bash
mkdir /mnt/data
echo 'Hello from Kubernetes storage' > /mnt/data/index.html
```

Create PersistentVolume:
```bash
kubectl create -f pods/storage/pv-volume.yaml
kubectl get pv task-pv-volume
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
task-pv-volume   1Gi        RWO            Retain           Available           manual                  5s
```

Now, create PersistentVolumeClaim:
```bash
kubectl create -f pods/storage/pv-claim.yaml
```

Check pv status
```bash
kubectl get pv task-pv-volume
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
task-pv-volume   1Gi        RWO            Retain           Bound    default/task-pv-claim   manual                  3m44s
```

Get PersistentVolumeClaim info:
```bash
kubectl get pvc task-pv-claim
NAME            STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   AGE
task-pv-claim   Bound    task-pv-volume   1Gi        RWO            manual         72s
```

Create a pod which uses this PersistentVolumeClaim. From the Pod’s point of view, the claim is a volume.
```bash
kubectl create -f pods/storage/pv-pod.yaml
```

Verify pod:
```bash
kubectl get pod task-pv-pod
NAME          READY   STATUS    RESTARTS   AGE
task-pv-pod   1/1     Running   0          2m46s
```

Get a shell to the Container running in your Pod:
```bash
kubectl exec -it task-pv-pod -- /bin/bash
```

Verify that nginx is running.
```bash
apt-get update
apt-get install curl
curl localhost
```

Remove all:
```bash
kubectl delete pod task-pv-pod
kubectl delete pvc task-pv-claim
kubectl delete pv task-pv-volume
```

## Configure a Pod to Use a Projected Volume for Storage

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/configure-projected-volume-storage/)

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

## Configure a Security Context for a Pod or Container

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)


### Set the security context for a Pod

Create the Pod:
```bash
kubectl create -f pods/security/security-context.yaml
pod/security-context-demo created
```

It's running
```bash
kubectl get pod security-context-demo
NAME                    READY   STATUS    RESTARTS   AGE
security-context-demo   1/1     Running   0          39s
```

Confirm that UID is 1000:
```bash
kubectl exec -it security-context-demo -- ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
1000         1  0.0  0.0   4336   760 ?        Ss   05:27   0:00 /bin/sh -c node
1000         7  0.0  0.3 772124 22776 ?        Sl   05:27   0:00 node server.js
1000        20  0.0  0.0  17500  2052 pts/0    Rs+  05:28   0:00 ps aux
```

Verify that volumes are using fsGroup GID
```bash
kubectl exec -it security-context-demo -- ls -l /data
total 4
drwxrwsrwx 2 root 2000 4096 Feb 25 05:27 demo
```

Create a testFile:
```bash
kubectl exec -it security-context-demo -- sh
cd /data/demo

ls -l
total 4
-rw-r--r-- 1 1000 2000 6 Feb 25 05:37 test-file

exit
```

### Set the security context for a Container

Security settings that you specify for a Container apply only to the individual Container, and they override settings made at the Pod level when there is overlap.

Create a Pod:
```bash
kubectl create -f pods/security/security-context-2.yaml
```

Verify it:
```bash
kubectl get pod security-context-demo-2
NAME                      READY   STATUS    RESTARTS   AGE
security-context-demo-2   1/1     Running   0          32s
```

Check that Pod is using UID 2000:
```bash
kubectl exec -it security-context-demo-2 -- ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
2000         1  0.0  0.0   4336   756 ?        Ss   05:43   0:00 /bin/sh -c node
2000         8  0.0  0.3 772124 22680 ?        Sl   05:43   0:00 node server.js
2000        13  0.0  0.0  17500  2112 pts/0    Rs+  05:48   0:00 ps aux
```

### Set capabilities for a Container

Create a Pod without capabilities:
```bash
kubectl create -f pods/security/security-context-3.yaml
```

UID is root's one
```bash
kubectl exec -it security-context-demo-3 -- ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   4336   724 ?        Ss   05:51   0:00 /bin/sh -c node
root         8  0.1  0.3 772124 22780 ?        Sl   05:51   0:00 node server.js
root        13  0.0  0.0  17500  2112 pts/0    Rs+  05:51   0:00 ps aux
```

Create a pod with capabilities:
```bash
kubectl create -f pods/security/security-context-4.yaml
```

## Configure Service Accounts for Pods

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

### Use the Default Service Account to access the API server

By default, servie account is 'default'

```bash
kubectl get pods redis -o yaml | grep serviceAccountName
  serviceAccountName: default
```

### Use Multiple Service Accounts

Get service accounts:
```bash
kubectl get serviceAccounts
NAME                                    SECRETS   AGE
default                                 1         2d19h
nginx-ingress-microk8s-serviceaccount   1         2d19h
```

Create serviceAccount:
```bash
kubectl create -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
EOF
```

Check serviceAccount:
```bash
kubectl get serviceaccounts/build-robot -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2019-02-25T16:40:02Z"
  name: build-robot
  namespace: default
  resourceVersion: "2208326"
  selfLink: /api/v1/namespaces/default/serviceaccounts/build-robot
  uid: ff37cb4a-391b-11e9-b481-5254000baa03
secrets:
- name: build-robot-token-pgkl7
```

To use a non-default service account, simply set the spec.serviceAccountName field of a pod to the name of the service account you wish to use.

Delete service account:
```bash
kubectl delete serviceaccount/build-robot
```

### Manually create a service account API token.

Create service account again:
```bash
kubectl create -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
EOF
```

Create a secret:
```bash
kubectl create -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: build-robot-secret
  annotations:
    kubernetes.io/service-account.name: build-robot
type: kubernetes.io/service-account-token
EOF
```

Get your secret
```bash
kubectl describe secrets/build-robot-secret
Name:         build-robot-secret
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: build-robot
              kubernetes.io/service-account.uid: 2b208294-391d-11e9-b481-5254000baa03

Type:  kubernetes.io/service-account-token

Data
====
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImJ1aWxkLXJvYm90LXNlY3JldCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJidWlsZC1yb2JvdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjJiMjA4Mjk0LTM5MWQtMTFlOS1iNDgxLTUyNTQwMDBiYWEwMyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmJ1aWxkLXJvYm90In0.bBSFqzHXQRSYeP3QykI23_rifQ_OnO5oajN1jjGE8-1GZpOHgoMc6NHSAFqkboMqf4SFGmE5FP8zdncmQk8e6CVTR_W2ayOfqU0sIzt85LHs65XFgpHyu64w0ouqZ9mXcegxFSLNIyelWe2CuEEkZqreuhSMjwEvnNz8XgM5Zj013k9ZvnQZ38dczwVvEuanoaAZSMEVVh8KgijDTlzDMaAUdHWFGjcncuI8Zw9uplCS_5BFrKtOEGWVSq5tVnyPlHwtR8a2gujAcLJ-DHD3-d0lsNeuWD6v4wSt0UrygIV7aX-tTk7OPxJW2bsCyOPzVz4O2mwFAezERmZpkX89dQ
ca.crt:     1094 bytes
```

### Add ImagePullSecrets to a service account

Create a secret:
```bash
kubectl create secret docker-registry myregistrykey --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL
```

Get secret:
```bash
kubectl get secrets myregistrykey
NAME            TYPE                             DATA   AGE
myregistrykey   kubernetes.io/dockerconfigjson   1      4m30s
```

Modify the default service account for the namespace to use this secret as an imagePullSecret.
```bash
kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "myregistrykey"}]}'
```

## Configure Liveness and Readiness Probes

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)


### Define a liveness command

Create the pod:
```bash
kubectl create -f pods/probe/exec-liveness.yaml
```

Check that continer is killed:
```bash
kubectl describe pod liveness-exec

  Type     Reason     Age                From                             Message
  ----     ------     ----               ----                             -------
  Normal   Scheduled  86s                default-scheduler                Successfully assigned default/liveness-exec to k8.daedalus-project.io
  Warning  Unhealthy  36s (x3 over 46s)  kubelet, k8.daedalus-project.io  Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Pulling    5s (x2 over 84s)   kubelet, k8.daedalus-project.io  pulling image "k8s.gcr.io/busybox"
  Normal   Killing    5s                 kubelet, k8.daedalus-project.io  Killing container with id docker://liveness:Container failed liveness probe.. Container will be killed and recreated.
  Normal   Pulled     4s (x2 over 82s)   kubelet, k8.daedalus-project.io  Successfully pulled image "k8s.gcr.io/busybox"
  Normal   Created    4s (x2 over 81s)   kubelet, k8.daedalus-project.io  Created container
  Normal   Started    3s (x2 over 81s)   kubelet, k8.daedalus-project.io  Started container
```

Check http Liveness
```bash
kubectl create -f pods/probe/http-liveness.yaml
```

```bash
kubectl describe pod liveness-http

Events:
  Type     Reason     Age               From                             Message
  ----     ------     ----              ----                             -------
  Normal   Scheduled  53s               default-scheduler                Successfully assigned default/liveness-http to k8.daedalus-project.io
  Warning  Unhealthy  5s (x6 over 32s)  kubelet, k8.daedalus-project.io  Liveness probe failed: HTTP probe failed with statuscode: 500
  Normal   Pulling    4s (x3 over 46s)  kubelet, k8.daedalus-project.io  pulling image "k8s.gcr.io/liveness"
  Normal   Killing    4s (x2 over 25s)  kubelet, k8.daedalus-project.io  Killing container with id docker://liveness:Container failed liveness probe.. Container will be killed and recreated.
  Normal   Pulled     3s (x3 over 45s)  kubelet, k8.daedalus-project.io  Successfully pulled image "k8s.gcr.io/liveness"
  Normal   Created    2s (x3 over 44s)  kubelet, k8.daedalus-project.io  Created container
  Normal   Started    2s (x3 over 44s)  kubelet, k8.daedalus-project.io  Started container
```

Define tcp liveness
```bash
kubectl create -f pods/probe/tcp-liveness-readiness.yaml
```

```bash
kubectl get pod goproxy --watch
NAME      READY   STATUS    RESTARTS   AGE
goproxy   0/1     Running   0          6s
goproxy   1/1   Running   0     13s
```

## Assign Pods to Nodes

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/)

### Add a label to a node

Get nodes:
```bash
kubectl get nodes

NAME                           STATUS                     ROLES    AGE     VERSION
kubernetes-master              Ready,SchedulingDisabled   <none>   3m43s   v1.13.3
kubernetes-minion-group-9g79   Ready                      <none>   3m34s   v1.13.3
kubernetes-minion-group-r7st   Ready                      <none>   3m33s   v1.13.3
kubernetes-minion-group-s881   Ready                      <none>   3m40s   v1.13.3
```

Chose one of your nodes, and add a label to it:
```bash
kubectl label nodes kubernetes-minion-group-9g79 disktype=ssd
```

Check label:
```bash
kubectl get nodes --show-labels | grep disk
kubernetes-minion-group-9g79   Ready                      <none>   5m38s   v1.13.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/fluentd-ds-ready=true,beta.kubernetes.io/instance-type=n1-standard-2,beta.kubernetes.io/os=linux,disktype=ssd,failure-domain.beta.kubernetes.io/region=us-central1,failure-domain.beta.kubernetes.io/zone=us-central1-b,kubernetes.io/hostname=kubernetes-minion-group-9g79
```

### Create a pod that gets scheduled to your chosen node

Create the pod:
```bash
kubectl create -f pods/pod-nginx.yaml
```

Verify that the pod is running on your chosen node:
```bash
kubectl get pods --output=wide
NAME    READY   STATUS    RESTARTS   AGE   IP          NODE                           NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          35s   10.64.2.5   kubernetes-minion-group-9g79   <none>           <none>
```

## Configure Pod Initialization

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/)

### Create a Pod that has an Init Container

Create the pod:
```bash
kubectl create -f pods/init-containers.yaml
```

Get pods:
```bash
kubectl get pod init-demo
```

Enter the pod and perform a curl request:
```bash
kubectl exec -it init-demo -- /bin/bash
apt-get update
apt-get install curl
curl localhost
```

## Attach Handlers to Container Lifecycle Events

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/)

### Define postStart and preStop handlers

Create the pod:
```bash
kubectl create -f pods/lifecycle-events.yaml
```

Verify that PostStart command worked:
```bash
kubectl exec -it lifecycle-demo -- cat /usr/share/message
Hello from the postStart handler
```

## Configure a Pod to Use a ConfigMap
[Link](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

### Create a ConfigMap

Get files:
```bash
mkdir -p configure-pod-container/configmap/kubectl/
wget https://k8s.io/docs/tasks/configure-pod-container/configmap/kubectl/game.properties -O configure-pod-container/configmap/kubectl/game.properties
wget https://k8s.io/docs/tasks/configure-pod-container/configmap/kubectl/ui.properties -O configure-pod-container/configmap/kubectl/ui.properties
```

Create config map from files:
```bash
kubectl create configmap game-config --from-file=configure-pod-container/configmap/kubectl/
```

Verify configmap
```bash
kubectl describe configmaps game-config

Name:         game-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
ui.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

Events:  <none>
```

### Create ConfigMaps from files

```bash
kubectl create configmap game-config-2 --from-file=configure-pod-container/configmap/kubectl/game.properties
kubectl describe configmaps game-config-2
Name:         game-config-2
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
Events:  <none>
```

Create a ConfigMap from an env-file
```bash
kubectl create configmap game-config-env-file --from-env-file=configure-pod-container/configmap/kubectl/game-env-file.properties

kubectl get configmap game-config-env-file -o yaml
apiVersion: v1
data:
  allowed: '"true"'
  enemies: aliens
  lives: "3"
kind: ConfigMap
metadata:
  creationTimestamp: "2019-02-26T07:05:39Z"
  name: game-config-env-file
  namespace: default
  resourceVersion: "2271866"
  selfLink: /api/v1/namespaces/default/configmaps/game-config-env-file
  uid: ec81b486-3994-11e9-a452-5254000baa03
```

When passing --from-env-file multiple times to create a ConfigMap from multiple data sources, only the last env-file is used:

### Define the key to use when creating a ConfigMap from a file

```bash
kubectl create configmap game-config-3 --from-file=game-special-key=configure-pod-container/configmap/kubectl/game.properties
kubectl get configmaps game-config-3 -o yaml

kubectl get configmaps game-config-3 -o yaml
apiVersion: v1
data:
  game-special-key: |-
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
kind: ConfigMap
metadata:
  creationTimestamp: "2019-02-26T07:07:56Z"
  name: game-config-3
  namespace: default
  resourceVersion: "2272031"
  selfLink: /api/v1/namespaces/default/configmaps/game-config-3
  uid: 3e1df533-3995-11e9-a452-5254000baa03
```

### Create ConfigMaps from literal values

```bash
kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm
kubectl get configmaps special-config -o yaml

apiVersion: v1
data:
  special.how: very
  special.type: charm
kind: ConfigMap
metadata:
  creationTimestamp: "2019-02-26T07:15:07Z"
  name: special-config
  namespace: default
  resourceVersion: "2272560"
  selfLink: /api/v1/namespaces/default/configmaps/special-config
  uid: 3f003906-3996-11e9-a452-5254000baa03
```

### Define container environment variables using ConfigMap data

```bash
kubectl delete configmap special-config
kubectl create configmap special-config --from-literal=special.how=very
```

Create the pod:
```bash
kubectl create -f pods/dapi-test-pod.yaml
```

### Define container environment variables with data from multiple ConfigMaps

Define env-config configmap:
```bash
kubectl create configmap env-config --from-literal=log_level=INFO
```

Create the pod:
```bash
kubectl create -f pods/dapi-test-pod-env.yaml
```

Get output:
```bash
k logs dapi-test-pod-env | grep SPECIAL_LEVEL_KEY
SPECIAL_LEVEL_KEY=very
```

### Use ConfigMap-defined environment variables in Pod commands

Create the pod:
```bash
k create -f pods/dapi-test-pod-command.yaml
```

Get output:
```bash
k logs  dapi-test-pod-command
thespecialLevel specialType
```

### Add ConfigMap data to a Volume

Create the pod:
```bash
k create -f pods/dapi-test-pod-volume.yaml
```

Get /etc/config
```bash
k logs  dapi-test-pod-volume
SPECIAL_LEVEL
SPECIAL_TYPE
special.how
special.level
special.type
```

### Add ConfigMap data to a specific path in the Volume

Create the pod:
```bash
k logs dapi-test-configs
very
```

## Share Process Namespace between Containers in a Pod

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/share-process-namespace/)

This is a beta feature.

Create the pod:
```bash
k create -f pods/share-process-namespace.yaml
```

## Translate a Docker Compose File to Kubernetes Resources

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/translate-compose-kubernetes/)

Install [kompose](http://kompose.io/).

Deploy
```bash
 kompose up
  We are going to create Kubernetes Deployments, Services and PersistentVolumeClaims for your Dockerized application.
  If you need different kind of resources, use the 'kompose convert' and 'kubectl create -f' commands instead.

  INFO Successfully created Service: redis          
  INFO Successfully created Service: web            
  INFO Successfully created Deployment: redis       
  INFO Successfully created Deployment: web         

  Your application has been deployed to Kubernetes. You can run 'kubectl get deployment,svc,pods,pvc' for details.
```

Check
```bash
kubectl get deployment,svc,pods,pvc
NAME                                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/frontend               1/1     1            1           2m39s
deployment.extensions/redis-master           1/1     1            1           2m39s
deployment.extensions/redis-slave            1/1     1            1           2m39s

NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/frontend               LoadBalancer   10.152.183.43    <pending>     80:30998/TCP   2m39s
service/redis-master           ClusterIP      10.152.183.242   <none>        6379/TCP       2m39s
service/redis-slave            ClusterIP      10.152.183.144   <none>        6379/TCP       2m39s

NAME                                          READY   STATUS             RESTARTS   AGE
pod/frontend-784f75ddb7-wtg5f                 1/1     Running            0          2m39s
pod/redis-master-97979696c-wbmhg              1/1     Running            0          2m39s
pod/redis-slave-6fd879d46c-9srqx              1/1     Running            0          2m39s
```

Convert docker composer to kubernetes file:
```bash
kompose convert
  INFO Kubernetes file "frontend-service.yaml" created
  INFO Kubernetes file "redis-master-service.yaml" created
  INFO Kubernetes file "redis-slave-service.yaml" created
  INFO Kubernetes file "frontend-deployment.yaml" created
  INFO Kubernetes file "redis-master-deployment.yaml" created
  INFO Kubernetes file "redis-slave-deployment.yaml" created
```
