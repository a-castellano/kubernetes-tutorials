# Inject Data Into Applications

## Define a Command and Arguments for a Container

[Link](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/)

Define a command and arguments when you create a Pod
```bash
k create -f pods/commands.yaml
```

Get logs:
```bash
k logs command-demo

command-demo
tcp://10.0.0.1:443
```

Use environment variables to define arguments
```bash
k create -f pods/commands-env-variables.yaml
```

Get logs
```bash
k logs command-demo-env-variables
hello world
```

Run a command in a shell
```bash
k create -f pods/commands-shell.yaml
```

Get logs:
```bash
k logs command-demo-shell
hello
hello

k logs command-demo-shell
hello
hello
hello
hello
```

## Define Environment Variables for a Container

[Link](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)

Define an environment variable for a container
```bash
k create -f pods/inject/envars.yaml
```

List the running Pods
```bash
kubectl get pods -l purpose=demonstrate-envars
NAME         READY   STATUS    RESTARTS   AGE
envar-demo   1/1     Running   0          37s
```

Get shell
```bash
k exec -it envar-demo -- /bin/bash
root@envar-demo:/# printenv
NODE_VERSION=4.4.2
HOSTNAME=envar-demo
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT=tcp://10.0.0.1:443
TERM=xterm
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_HOST=10.0.0.1
NPM_CONFIG_LOGLEVEL=info
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
DEMO_FAREWELL=Such a sweet sorrow
SHLVL=1
HOME=/root
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP_ADDR=10.0.0.1
KUBERNETES_PORT_443_TCP=tcp://10.0.0.1:443
DEMO_GREETING=Hello from the environment
_=/usr/bin/printenv
```

## Expose Pod Information to Containers Through Environment Variables

```bash
k create -f pods/inject/dapi-envars-pod.yaml
```

Get logs:
```bash
k logs dapi-envars-fieldref

kubernetes-minion-group-cg12
dapi-envars-fieldref
default
10.64.1.6
default
```

Show env variables:
```bash
k exec -it dapi-envars-fieldref -- printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=dapi-envars-fieldref
TERM=xterm
MY_POD_NAMESPACE=default
MY_POD_IP=10.64.1.6
MY_POD_SERVICE_ACCOUNT=default
MY_NODE_NAME=kubernetes-minion-group-cg12
MY_POD_NAME=dapi-envars-fieldref
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.0.0.1
KUBERNETES_SERVICE_HOST=10.0.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.0.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.0.0.1:443
HOME=/root
```

Use Container fields as values for environment variables
```bash
k create -f pods/inject/dapi-envars-container.yaml
```

Get logs:
```bash
kubectl logs dapi-envars-resourcefieldref

1
1
33554432
67108864
```

## Expose Pod Information to Containers Through Files

[Link](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/)

Create the pod:
```bash
k create -f pods/inject/dapi-volume.yaml
```

Get logs:
```bash
k logs kubernetes-downwardapi-volume-example


cluster="test-cluster1"
rack="rack-22"
zone="us-est-coast"

build="two"
builder="john-doe"
kubernetes.io/config.seen="2019-03-02T10:57:03.219556821Z"
kubernetes.io/config.source="api"
kubernetes.io/limit-ranger="LimitRanger plugin set: cpu request for container client-container"
```

Check /etc/podinfo
```bash
k exec -it kubernetes-downwardapi-volume-example -- cat /etc/podinfo/labels

cluster="test-cluster1"
rack="rack-22"

k exec -it kubernetes-downwardapi-volume-example --  cat /etc/podinfo/annotations
build="two"
builder="john-doe"
kubernetes.io/config.seen="2019-03-02T10:57:03.219556821Z"
kubernetes.io/config.source="api"
kubernetes.io/limit-ranger="LimitRanger plugin set: cpu request for container client-container"

k exec -it kubernetes-downwardapi-volume-example --  ls -laR /etc/podinfo
/etc/podinfo:
total 4
drwxrwxrwt    3 root     root           120 Mar  2 10:57 .
drwxr-xr-x    1 root     root          4096 Mar  2 10:57 ..
drwxr-xr-x    2 root     root            80 Mar  2 10:57 ..2019_03_02_10_57_03.724308127
lrwxrwxrwx    1 root     root            31 Mar  2 10:57 ..data -> ..2019_03_02_10_57_03.724308127
lrwxrwxrwx    1 root     root            18 Mar  2 10:57 annotations -> ..data/annotations
lrwxrwxrwx    1 root     root            13 Mar  2 10:57 labels -> ..data/labels

/etc/podinfo/..2019_03_02_10_57_03.724308127:
total 8
drwxr-xr-x    2 root     root            80 Mar  2 10:57 .
drwxrwxrwt    3 root     root           120 Mar  2 10:57 ..
-rw-r--r--    1 root     root           219 Mar  2 10:57 annotations
-rw-r--r--    1 root     root            58 Mar  2 10:57 labels
```

Store Container fields
```bash
k create -f  pods/inject/dapi-volume-resources.yaml
```

```bash
k exec -it kubernetes-downwardapi-volume-example-2 -- cat /etc/podinfo/cpu_limit
250
```

```bash
k logs kubernetes-downwardapi-volume-example-2


250
125
64
32
```

Distribute Credentials Securely Using Secrets

[Link](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)

Create a secret:
```bash
k create -f pods/inject/secret.yaml
```

Show secret
```bash
k get secret test-secret
NAME          TYPE     DATA   AGE
test-secret   Opaque   2      167m
```

View detailed info:
```bash
k describe secret test-secret

Name:         test-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  12 bytes
username:  6 bytes
```

Secret can be created from literal too:
```bash
k create secret generic test-secret --from-literal=username='my-app' --from-literal=password='39528$vdg7Jb'
```

Create pod containing secrets:
```bash
k create -f pods/inject/secret-pod.yaml
```

Show secrets:
```bash
k exec -it secret-test-pod -- cat /etc/secret-volume/username
my-app

k exec -it secret-test-pod -- cat /etc/secret-volume/password
39528$vdg7Jb
```

Use files too:
```bash
k create secret generic test-secret --from-literal=username='my-app' --from-literal=password='39528$vdg7Jb' --from-file=ssh-publickey=pods/inject/secret_files/public_key --from-file=ssh-privatekey=pods/inject/secret_files/private_key
```

Check new secrets:
```bash
k exec -it secret-test-pod -- ls -l /etc/secret-volume/
total 0
lrwxrwxrwx 1 root root 15 Mar  2 15:02 password -> ..data/password
lrwxrwxrwx 1 root root 21 Mar  2 15:02 ssh-privatekey -> ..data/ssh-privatekey
lrwxrwxrwx 1 root root 20 Mar  2 15:02 ssh-publickey -> ..data/ssh-publickey
lrwxrwxrwx 1 root root 15 Mar  2 15:02 username -> ..data/username
```

Create a Pod that has access to the secret data through environment variables:
```bash
k create -f pods/inject/secret-envars-pod.yaml
```

Get secrets:
```bash
kubectl exec -it secret-envars-test-pod -- printenv | grep SECRET
SECRET_USERNAME=my-app
SECRET_PASSWORD=39528$vdg7Jb
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
