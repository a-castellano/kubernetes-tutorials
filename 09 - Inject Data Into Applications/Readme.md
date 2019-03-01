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
