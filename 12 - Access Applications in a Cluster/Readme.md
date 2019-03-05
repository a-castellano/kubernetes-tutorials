# Access Applications in a Cluster

## Use Port Forwarding to Access Applications in a Cluster

[Link](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

Create a Redis deployment:
```bash
k create -f application/guestbook/redis-master-deployment.yaml
```

Create a Redis service:
```bash
k create -f application/guestbook/redis-master-service.yaml
```

Get pods:
```bash
k get pods -l app=redis
NAME                            READY   STATUS    RESTARTS   AGE
redis-master-6fbbc44567-64gmq   1/1     Running   0          4m13s
```

Forward port
```bash
k port-forward redis-master-6fbbc44567-64gmq 6380:6379

Forwarding from 127.0.0.1:6380 -> 6379
Forwarding from [::1]:6380 -> 6379
```

It seems to be working
```bash
redis-cli -p 6380
127.0.0.1:6380> ping
PONG
127.0.0.1:6380>
```

## Use a Service to Access an Application in a Cluster
[Link](https://kubernetes.io/docs/tasks/access-application-cluster/service-access-application-cluster/)

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
