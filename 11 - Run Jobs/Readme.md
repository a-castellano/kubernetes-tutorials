# Run Jobs

## Running Automated Tasks with a CronJob

[Link](https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/)

Create cronjob:
```bash
k create -f application/job/cronjob.yaml
```

Show cronjobs:
```bash
k get cronjob hello

NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        <none>          28s

NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     1        14s             36s

NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        20s             42s

NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        21s             43s
```

Delete cron
```bash
k delete cronjob hello
```

Keep more pods per executed tasks
```bash
k create -f application/job/cronjobHistoryLimit.yaml
```

Check she number of completed pods after 6 minutes:

Delete cronjob again
```bash
k delete cronjob hello
```

## Parallel Processing using Expansions

[Link](https://kubernetes.io/docs/tasks/job/parallel-processing-expansion/)

Create jobs from template
```bash
for i in apple banana cherry; do   cat application/job/job-tmpl.yaml | sed "s/\$ITEM/$i/" > ./jobs/job-$i.yaml; done
```

Create jobs
```bash
k create -f ./jobs
```

Check termination
```bash
k get jobs -l jobgroup=jobexample

NAME                  COMPLETIONS   DURATION   AGE
process-item-apple    1/1           16s        27s
process-item-banana   1/1           18s        25s
process-item-cherry   1/1           17s        25s
```

Chack output
```bash
for p in $(k get pods -l jobgroup=jobexample -o name); do k logs $p; done
Processing item apple
Processing item banana
Processing item cherry
```

Define render template alias
```bash
alias render_template='python -c "from jinja2 import Template; import sys; print(Template(sys.stdin.read()).render());"'
```

Create new jobs:
```bash
cat application/job/job.yaml.jinja2 | render_template | k create -f -

job.batch/jobexample-apple created
job.batch/jobexample-banana created
job.batch/jobexample-raspberry created
```

## Coarse Parallel Processing Using a Work Queue

[Link](https://kubernetes.io/docs/tasks/job/coarse-parallel-processing-work-queue/)

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
