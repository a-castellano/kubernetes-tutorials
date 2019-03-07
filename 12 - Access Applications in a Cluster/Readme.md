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

Run a Hello World application in your cluster:
```bash
k run hello-world --replicas=2 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0  --port=8080
```

Create a Service object that exposes the deployment:
```bash
k expose deployment hello-world --type=NodePort --name=example-service
```

Show service:
```bash
k describe services example-service
```

List the pods that are running the Hello World application:
```bash
k get pods --selector="run=load-balancer-example" --output=wide

NAME                           READY   STATUS    RESTARTS   AGE     IP          NODE                           NOMINATED NODE   READINESS GATES
hello-world-696b6b59bd-wvw69   1/1     Running   0          5m29s   10.64.3.3   kubernetes-minion-group-6kpw   <none>           <none>
hello-world-696b6b59bd-ztgd9   1/1     Running   0          5m29s   10.64.2.4   kubernetes-minion-group-rkxp   <none>           <none>
```

Check node port
```bash
kubectl describe services example-service
Name:                     example-service
Namespace:                default
Labels:                   run=load-balancer-example
Annotations:              <none>
Selector:                 run=load-balancer-example
Type:                     NodePort
IP:                       10.0.196.62
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30956/TCP
Endpoints:                10.64.2.4:8080,10.64.3.3:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Check webapp
```bash
curl http://35.193.51.51:30956/
Hello Kubernetes!
```

## Connect a Front End to a Back End Using a Service

[Link](https://kubernetes.io/docs/tasks/access-application-cluster/connecting-frontend-backend/)

Create the backend Deployment:
```bash
k create -f service/access/hello.yaml
```

Create service:
```bash
k create -f service/access/hello-service.yaml
```

Create the frontend
```bash
k create -f service/access/frontend.yaml
```

## Set up Ingress on Minikube with the NGINX Ingress Controller

[Link](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/)

Verify that the NGINX Ingress controller is running
```bash
k get pods | grep nginx
nginx-ingress-microk8s-controller-njzmh   1/1     Running   0          34h
```

### Deploy a hello, world app

Create a Deployment using the following command:
```bash
k run web --image=gcr.io/google-samples/hello-app:1.0 --port=8080
```

Expose the Deployment:
```bash
k expose deployment web --target-port=8080 --type=NodePort
```

Check:
```bash
curl -IL YOUR_IP:PORT
HTTP/1.1 200 OK
Date: Thu, 07 Mar 2019 05:03:58 GMT
Content-Length: 60
Content-Type: text/plain; charset=utf-8
```

```bash
k create -f ingress/example-ingress.yaml
```

Verify ingress:
```bash
kubectl get ingress
NAME              HOSTS              ADDRESS     PORTS   AGE
example-ingress   hello-world.info   127.0.0.1   80      66s
```

```bash
curl hello-world.info
Hello, world!
Version: 1.0.0
Hostname: web-77656d79f8-5rtzk
```

Create Second Deployment
```bash
k run web2 --image=gcr.io/google-samples/hello-app:2.0 --port=8080
```

Expose deployment:
```bash
k expose deployment web2 --target-port=8080 --type=NodePort
```

Update ingress
```bash
k apply -f ingress/example-ingress2.yaml
```

Check
```bash
curl hello-world.info
Hello, world!
Version: 1.0.0
Hostname: web-77656d79f8-5rtzk
```

```bash
curl hello-world.info/v2
Hello, world!
Version: 2.0.0
Hostname: web2-675cf6d7b9-pk7g5
```

## Communicate Between Containers in the Same Pod Using a Shared Volume

[Link](https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/)

Creating a Pod that runs two Containers
```bash
k create -f pods/two-container-pod.yaml
```

Debian container only creates index.html
```bash
kubectl get pod two-containers --output=yaml

  containerStatuses:
  - containerID: docker://e029c030bb23672d9889f5fe812622868228a0c27fbf55593261c60184a5eea1
    image: debian:latest
    imageID: docker-pullable://debian@sha256:72e996751fe42b2a0c1e6355730dc2751ccda50564fec929f76804a6365ef5ef
    lastState: {}
    name: debian-container
    ready: false
    restartCount: 0
    state:
      terminated:
        containerID: docker://e029c030bb23672d9889f5fe812622868228a0c27fbf55593261c60184a5eea1
        exitCode: 0
        finishedAt: "2019-03-07T05:42:24Z"
        reason: Completed
        startedAt: "2019-03-07T05:42:24Z"
  - containerID: docker://c349532f85bcd67483efe446801cfeb760b588808e6e8462b36e391efbd09f69
    image: nginx:latest
    imageID: docker-pullable://nginx@sha256:98efe605f61725fd817ea69521b0eeb32bef007af0e3d0aeb6258c6e6fe7fc1a
    lastState: {}
    name: nginx-container
    ready: true
    restartCount: 0
    state:
      running:
        startedAt: "2019-03-07T05:42:16Z"
```

Get a shell to nginx Container:
```bash
k exec -it two-containers -c nginx-container -- /bin/bash

apt-get update
apt-get install curl procps
```

Check Nginx processes
```bash
ps aux

USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  32656  5232 ?        Ss   05:42   0:00 nginx: master process nginx -g daemon off;
nginx        8  0.0  0.0  33112  2480 ?        S    05:42   0:00 nginx: worker process
root         9  0.0  0.0  18136  3236 pts/0    Ss   05:46   0:00 /bin/bash
root      3675  0.0  0.0  36636  2816 pts/0    R+   05:49   0:00 ps aux
```

Check Nginx
```bash
curl localhost
Hello from the debian container
```
