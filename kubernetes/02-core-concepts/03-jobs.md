# Jobs

A Job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate. 

Aliases: `jobs` and `job`

- [Jobs](#jobs)
  - [Creation](#creation)
  - [Repeat until failure](#repeat-until-failure)
  - [Complete X times](#complete-x-times)
  - [Parallelize](#parallelize)
  - [Debugging Jobs](#debugging-jobs)


## Creation

Create a Job with command line:

```bash
k create job  pi-job -o yaml --dry-run=client --image=perl:5.34.0 >  pi.yaml
```

The `.spec.template` is a pod template and the only required field of the .spec.

The `RestartPolicy` can only be `Never` or `OnFailure`.


```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-job
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
...
```

## Repeat until failure

The `.spec.backoffLimit` specifies the number of retries before considering a Job as failed. The back-off limit is set by default to 6. 


```yaml
apiVersion: batch/v1
kind: Job
metadata:
  ...
spec:
  backoffLimit: 15 # The job retries 15 times before it quits
  template:
    spec:
      ...
```

## Complete X times

Use `.spec.completions` to 

```yaml
spec:
  completions: 3
  backoffLimit: 25 # This is so the job does not quit before it succeeds.
```

## Parallelize

Use `.spec.parallelism` to 

```yaml
spec:
  completions: 3
  backoffLimit: 25 # This is so the job does not quit before it succeeds.
```
## Debugging Jobs

As Jobs are run in a Pod, debugging is similar to pods. 

- get the job completion information
- describe the job. `Pods Statuses` lists when the Job fails and how many times.
- find the pod name and describe the pod
- if the pod created some content, check it with the logs
- to restart the job, it must be deleted first

```bash
$ kubectl get job my-job
NAME     COMPLETIONS   DURATION   AGE
my-job   0/1           22s        22s

$ kubectl describe job my-job
Name:             my-job
Namespace:        default
...
Parallelism:      1
Completions:      1
...
Pods Statuses:    0 Active (0 Ready) / 0 Succeeded / 5 Failed
...
Events:
  Type    Reason            Age    From            Message
  ----    ------            ----   ----            -------
  Normal  SuccessfulCreate  6m45s  job-controller  Created pod: my-job-wm9p9


$ kubectl get pods
NAME           READY   STATUS       RESTARTS   AGE
my-job-np4t2   0/1     StartError   0          6m32s
my-job-tb2kn   0/1     StartError   0          6m11s
my-job-vfvfd   0/1     StartError   0          5m41s
my-job-vm4z2   0/1     StartError   0          5m1s
my-job-wm9p9   0/1     StartError   0          61s

$ kubectl describe pods my-job-wm9p9
Name:             my-job-wm9p9
Namespace:        labs
...
Events:
  Type     Reason     Age    From      Message
  ----     ------     ----   ----      -------
  Normal   Pulling    1m25s  kubelet   Pulling image "busybox"
  Normal   Pulled     1m25s  kubelet   Successfully pulled image "busybox" in 609ms (609ms including waiting)
  Normal   Created    1m25s  kubelet   Created container my-job
  Warning  Failed     1m24s  kubelet   Error: failed to create containerd task: ....

$ kubectl logs my-job-wm9p9
...

$ kubectl delete job my-job 
job.batch "my-job" deleted
```

After fixing it, the status should be `Completed`.

```bash
$ kubectl get pods my-job-9nb88
NAME           READY   STATUS      RESTARTS   AGE
my-job-9nb88   0/1     Completed   0          3h

$ kubectl get job my-job 
NAME     COMPLETIONS   DURATION   AGE
my-job   1/1           82s        3h

$ kubectl describe job my-job 
Name:             my-job
...
Duration:         82s
Pods Statuses:    0 Active (0 Ready) / 1 Succeeded / 0 Failed
Pod Template:
...
```