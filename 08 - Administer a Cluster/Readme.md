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
