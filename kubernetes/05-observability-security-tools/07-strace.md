# Strace

Strace is a diagnostic, debugging and instructional userspace utility for Linux. It is used to monitor and tamper with interactions between processes and the Linux kernel, which include system calls, signal deliveries, and changes of process state. The operation of strace is made possible by the kernel feature known as ptrace.

There are Pods in Namespace team-yellow. A security investigation noticed that some processes running in these Pods are using the Syscall kill, which is forbidden by an internal policy of Team Yellow.

Find the offending Pod(s) and remove these by reducing the replicas of the parent Deployment to 0.

## Instructions

1. List the Pods in the `team-yellow` namespace.
2. Find the offending Pod(s) that are using the `kill` syscall.
3. Reduce the replicas of the parent Deployment to 0.
4. Verify that the Pods have been removed.

---

## Solution

1. List the Pods in the `team-yellow` namespace:

```bash
kubectl get pods -n team-yellow
```

2. Find the offending Pod(s) that are using the `kill` syscall:

```bash
kubectl exec -it <POD_NAME> -n team-yellow -- strace -p 1
```

3. Reduce the replicas of the parent Deployment to 0:

```bash
kubectl scale deployment <DEPLOYMENT_NAME> --replicas=0 -n team-yellow
```


candidate@cks7262:~$ k get pods -o wide -n team-yellow 
NAME                          READY   STATUS    RESTARTS   AGE   IP             NODE            NOMINATED NODE   READINESS GATES
collector1-784bb84d8-24ccx    1/1     Running   0          40d   10.244.1.87    cks7262-node1   <none>           <none>
collector1-784bb84d8-65lz2    1/1     Running   0          40d   10.244.1.93    cks7262-node1   <none>           <none>
collector2-7ff98dd886-565dg   1/1     Running   0          40d   10.244.1.152   cks7262-node1   <none>           <none>
collector3-854868d544-5d4sz   1/1     Running   0          40d   10.244.1.191   cks7262-node1   <none>           <none>
collector3-854868d544-vxwcc   1/1     Running   0          40d   10.244.1.69    cks7262-node1   <none>           <none>
candidate@cks7262:~$ ssh cks7262-node1
candidate@cks7262-node1:~$ sudo -i

root@cks7262-node1:~# sudo ps aux | grep -E 'collector[1-3]'
root       14668  0.0  0.0 702216   640 ?        Ssl  12:23   0:05 ./collector1-process
root       14743  0.0  0.0 702216   640 ?        Ssl  12:23   0:05 ./collector1-process
root       14778  0.0  0.1 702480  1536 ?        Ssl  12:23   0:06 ./collector3-process
root       14816  0.0  0.0 702224   512 ?        Ssl  12:23   0:05 ./collector2-process
root       14853  0.0  0.1 702480  1408 ?        Ssl  12:23   0:05 ./collector3-process
root@cks7262-node1:~# sudo strace -e kill -p  14668
strace: Process 14668 attached
^Cstrace: Process 14668 detached

root@cks7262-node1:~# sudo strace -e kill -p 14743
strace: Process 14743 attached
kill(666, SIGTERM)                      = -1 ESRCH (No such process)
--- SIGURG {si_signo=SIGURG, si_code=SI_TKILL, si_pid=1, si_uid=0} ---
--- SIGURG {si_signo=SIGURG, si_code=SI_TKILL, si_pid=1, si_uid=0} ---
--- SIGURG {si_signo=SIGURG, si_code=SI_TKILL, si_pid=1, si_uid=0} ---
--- SIGURG {si_signo=SIGURG, si_code=SI_TKILL, si_pid=1, si_uid=0} ---
--- SIGURG {si_signo=SIGURG, si_code=SI_TKILL, si_pid=1, si_uid=0} ---
--- SIGURG {si_signo=SIGURG, si_code=SI_TKILL, si_pid=1, si_uid=0} ---
^Cstrace: Process 14743 detached