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
