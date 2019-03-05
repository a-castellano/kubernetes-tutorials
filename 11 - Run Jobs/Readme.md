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

Start RabbitMQ as follows:
```bash
k create -f examples/celery-rabbitmq/rabbitmq-service.yaml
k create -f examples/celery-rabbitmq/rabbitmq-controller.yaml
```

Testing the message queue service
```bash
k run -i --tty temp --image ubuntu:18.04
apt-get update
apt-get install -y curl ca-certificates amqp-tools python dnsutils

export BROKER_URL=amqp://guest:guest@rabbitmq-service:5672
/usr/bin/amqp-declare-queue --url=$BROKER_URL -q foo -d
/usr/bin/amqp-publish --url=$BROKER_URL -r foo -p -b Hello
/usr/bin/amqp-consume --url=$BROKER_URL -q foo -c 1 cat && echo
```

Filling the Queue with tasks
```bash
/usr/bin/amqp-declare-queue --url=$BROKER_URL -q job1  -d
for f in apple banana cherry date fig grape lemon melon; do /usr/bin/amqp-publish --url=$BROKER_URL -r job1 -p -b $f; done
```

Create job
```bash
k create -f application/job/rabbitmq/job.yaml
```

## Fine Parallel Processing Using a Work Queue

[Link](https://kubernetes.io/docs/tasks/job/fine-parallel-processing-work-queue/)

Create redis pod and service:
```bash
k create -f application/job/redis/redis-pod.yaml
k create -f application/job/redis/redis-service.yaml
```

Filling the Queue with tasks
```bash
k run -i --tty temp --image redis --command "/bin/sh"
```

Fill queue:
```bash
# redis-cli -h redis
redis:6379> rpush job2 "apple"
(integer) 1
redis:6379> rpush job2 "banana"
(integer) 2
redis:6379> rpush job2 "cherry"
(integer) 3
redis:6379> rpush job2 "date"
(integer) 4
redis:6379> rpush job2 "fig"
(integer) 5
redis:6379> rpush job2 "grape"
(integer) 6
redis:6379> rpush job2 "lemon"
(integer) 7
redis:6379> rpush job2 "melon"
(integer) 8
redis:6379> rpush job2 "orange"
(integer) 9
redis:6379> lrange job2 0 -1
1) "apple"
2) "banana"
3) "cherry"
4) "date"
5) "fig"
6) "grape"
7) "lemon"
8) "melon"
9) "orange"
```

Create a job:
```bash
k create -f  application/job/redis/job.yaml
```

Check jobs:
```bash
kubectl describe jobs/job-wq-2
Name:           job-wq-2
Namespace:      default
Selector:       controller-uid=b59f4c88-3f07-11e9-970d-5254000baa03
Labels:         controller-uid=b59f4c88-3f07-11e9-970d-5254000baa03
                job-name=job-wq-2
Annotations:    <none>
Parallelism:    2
Completions:    <unset>
Start Time:     Tue, 05 Mar 2019 06:29:56 +0100
Completed At:   Tue, 05 Mar 2019 06:31:27 +0100
Duration:       91s
Pods Statuses:  0 Running / 2 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=b59f4c88-3f07-11e9-970d-5254000baa03
           job-name=job-wq-2
  Containers:
   c:
    Image:        acastellano/job-wq-2
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  117s  job-controller  Created pod: job-wq-2-ztbsl
  Normal  SuccessfulCreate  117s  job-controller  Created pod: job-wq-2-4cxv6
```

Check logs:
```bash
k logs job-wq-2-ztbsl
Worker with sessionID: 8f687746-8cbf-45b2-98fc-15bad1594442
Initial queue state: empty=False
Working on melon
Working on grape
Working on date
Working on banana
Waiting for work
Waiting for work
Waiting for work
Waiting for work
Waiting for work
Queue empty, exiting
```
