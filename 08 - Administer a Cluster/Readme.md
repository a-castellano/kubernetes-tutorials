# Administer a Cluster

## Manage Memory, CPU, and API Resources

### Configure Default Memory Requests and Limits for a Namespace

[Link](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)

Create a namespace
```bash
k create namespace default-mem-example
```

Create LimitRange object inside our namespace:
```bash
k create -f admin/resource/memory-defaults.yaml -n default-mem-example
```

Create a pod without defined limits inside _default-mem-example_ namespace
```bash
k create -f admin/resource/memory-defaults-pod.yaml -n default-mem-example
```

View detailed information about the Pod:
```bash
k get pod default-mem-demo --output=yaml -n default-mem-example

spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: default-mem-demo-ctr
    resources:
      limits:
        memory: 512Mi
      requests:
        memory: 256Mi
```

Create a container with limits but not requests:
```bash
k create -f admin/resource/memory-defaults-pod-2.yaml  -n default-mem-example
```

Check limits
```bash
k get pod default-mem-demo-2 --output=yaml -n default-mem-example

  containers:
  - image: nginx
    imagePullPolicy: Always
    name: default-mem-demo-2-ctr
    resources:
      limits:
        memory: 1Gi
      requests:
        memory: 1Gi
```
Requests should be defined too.

Create a container with requests but not limits:
```bash
k create -f admin/resource/memory-defaults-pod-3.yaml -n default-mem-example
```

Check limits:
```bash
k get pod default-mem-demo-3 --output=yaml -n default-mem-example

  containers:
  - image: nginx
    imagePullPolicy: Always
    name: default-mem-demo-3-ctr
    resources:
      limits:
        memory: 512Mi
      requests:
        memory: 128Mi
```
Limit value is namespace default limit.

### Configure Default CPU Requests and Limits for a Namespace

[Link](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/)

Create a namespace
```bash
k create namespace default-cpu-example
```

Create LimitRange object inside our namespace:
```bash
k create -f admin/resource/cpu-defaults.yaml -n default-cpu-example
```

Create a container without default limits neither requests.
```bash
k create -f admin/resource/cpu-defaults-pod.yaml -n default-cpu-example
```

Check limits:
```bash
k get pod default-cpu-demo --output=yaml -n default-cpu-example

spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: default-cpu-demo-ctr
    resources:
      limits:
        cpu: "1"
      requests:
        cpu: 500m
```

Specify limit but no requests
```bash
k create -f admin/resource/cpu-defaults-pod-2.yaml -n default-cpu-example
```

Check limits
```bash
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: default-cpu-demo-2-ctr
    resources:
      limits:
        cpu: "1"
      requests:
        cpu: "1"
```

I think that specifiying only limit (and get same request) is not a good idea.


Specify only requests:
```bash
k create -f admin/resource/cpu-defaults-pod-3.yaml -n default-cpu-example
```

Check limits:
```bash
k get pod default-cpu-demo-3 --output=yaml -n default-cpu-example

spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: default-cpu-demo-3-ctr
    resources:
      limits:
        cpu: "1"
      requests:
        cpu: 750m
```

### Configure Minimum and Maximum Memory Constraints for a Namespace

[Link](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-constraint-namespace/)

Create a namespace
```bash
k create namespace constraints-mem-example
```

Create a limit range (LimitRange):
```bash
k create -f admin/resource/memory-constraints.yaml -n constraints-mem-example
```

View detailed information about the LimitRange:
```bash
k get limitrange mem-min-max-demo-lr -n constraints-mem-example --output=yaml

apiVersion: v1
kind: LimitRange
metadata:
  creationTimestamp: "2019-02-28T15:57:57Z"
  name: mem-min-max-demo-lr
  namespace: constraints-mem-example
  resourceVersion: "2536630"
  selfLink: /api/v1/namespaces/constraints-mem-example/limitranges/mem-min-max-demo-lr
  uid: 9d9fb3f3-3b71-11e9-b54b-5254000baa03
spec:
  limits:
  - default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```

Create a container that satisfies the minimum and maximum memory constraints imposed by the LimitRange:
```bash
k create -f admin/resource/memory-constraints-pod.yaml -n constraints-mem-example
```

Pod is running:
```bash
k get pod constraints-mem-demo -n constraints-mem-example
NAME                   READY   STATUS    RESTARTS   AGE
constraints-mem-demo   1/1     Running   0          31s
```

Get pod info:
```bash
k get pod constraints-mem-demo --output=yaml -n constraints-mem-example

spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: constraints-mem-demo-ctr
    resources:
      limits:
        memory: 800Mi
      requests:
        memory: 600Mi
```

Delete your Pod:
```bash
k delete pod constraints-mem-demo -n constraints-mem-example
```

Attempt to create a Pod that exceeds the maximum memory constraint
```bash
k create -f admin/resource/memory-constraints-pod-2.yaml -n constraints-mem-example
Error from server (Forbidden): error when creating "admin/resource/memory-constraints-pod-2.yaml": pods "constraints-mem-demo-2" is forbidden: maximum memory usage per Container is 1Gi, but limit is 1536Mi.
```

Attempt to create a Pod that does not meet the minimum memory request
```bash
k create -f admin/resource/memory-constraints-pod-3.yaml -n constraints-mem-example
Error from server (Forbidden): error when creating "admin/resource/memory-constraints-pod-3.yaml": pods "constraints-mem-demo-3" is forbidden: minimum memory usage per Container is 500Mi, but request is 100Mi.
```

Create a Pod that does not specify any memory request or limits
```bash
k create -f admin/resource/memory-constraints-pod-4.yaml -n constraints-mem-example
```

It works, but.....check its properties:
```bash
k get pod constraints-mem-demo-4 --output=yaml -n constraints-mem-example

  containers:
  - image: nginx
    imagePullPolicy: Always
    name: constraints-mem-demo-4-ctr
    resources:
      limits:
        memory: 1Gi
      requests:
        memory: 1Gi
```

Delete the pod:
```bash
k delete pod constraints-mem-demo-4 -n constraints-mem-example
```

### Configure Minimum and Maximum CPU Constraints for a Namespace

[Link](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/)

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

## Advertise Extended Resources for a Node

[Link](https://kubernetes.io/docs/tasks/administer-cluster/extended-resource-node/)

Start proxy
```bash
kubectl proxy
```

Create dongle
```bash
curl --header "Content-Type: application/json-patch+json" --request PATCH --data '[{"op": "add", "path": "/status/capacity/example.com~1dongle", "value": "4"}]' http://localhost:8001/api/v1/nodes/k8.daedalus-project.io/status
```

Describe Node
```bash
kubectl describe node k8.daedalus-project.io

Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource            Requests         Limits
  --------            --------         ------
  cpu                 758m (18%)       498m (12%)
  memory              1343696Ki (22%)  1405136Ki (23%)
  ephemeral-storage   0 (0%)           0 (0%)
  example.com/dongle  0                0
```

Clean up
```bash
curl --header "Content-Type: application/json-patch+json" --request PATCH --data '[{"op": "remove", "path": "/status/capacity/example.com~1dongle"}]' http://localhost:8001/api/v1/nodes/k8.daedalus-project.io/status
```

Check clean
```bash
kubectl describe node k8.daedalus-project.io | grep dongle
```
