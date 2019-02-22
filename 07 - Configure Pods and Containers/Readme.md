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

#### Guaranteed

For a Pod to be given a QoS class of Guaranteed:

* Every Container in the Pod must have a memory limit and a memory request, and they must be the same.
* Every Container in the Pod must have a CPU limit and a CPU request, and they must be the same.

Create a nampespace
```bash
kubectl create namespace qos-example
```

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

```bash
```
