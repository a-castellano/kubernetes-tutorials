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

```

## Assign CPU Resources to Containers and Pods

[Link](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)
https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/

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
