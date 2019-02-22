# Administer a Cluster

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
