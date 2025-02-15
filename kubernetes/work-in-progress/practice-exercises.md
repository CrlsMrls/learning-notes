# Practice exercises

## Namespaces and nodes
- List all nodes' IPs and K8s version on each node
- Create a namespace
- Create a pod to that namespace
- Create a new pod to that namespace, with different label
- Select all pods in all namespaces, select pods within a namespace, select pods with a label

## Pods
- Create a nginx pod, prefilling the index.html with hostname and creation date
- Create a deployment with 3 nginx pods, accept traffic only after the `/tmp/healthy` file is created
- Create the file in these pods
- modify liveness probes delays and periods

## Jobs
- Create a job that sleeps for the first 15 seconds
- Create a job that must complete 3 times
- Create a job that fails and it retries 5 times
- Create a job that fails and it retries 10 times, paralleled

## Assign pods to nodes
- Assign pod to a node
- Create a more complex rule to assign a pod to a node
- Make a node to run only similar pods
- Distribute four pods of two types in two nodes (1 front and 1 backend per node)

## Deployments
- Create a deployment called `web` with `nginx:1.17` image and 5 replicas
  - `busybox` init container prints `hostname` and system info (`uname -a`) into `/usr/share/nginx/html/index.html`
  - upgrade to `nginx:1.24-alpine` and new tag `v2` with rolling update
  - check history, export both versions' history to file `one.out` and `two.out`, `diff` both files.
  - upgrade to an invalid version `nginx:91.1`
  - roll back 
  - increase replicas to 3
  - autoscale pods between 5 to 8 with CPU utilization at 80%
  - upgrade to `nginx:1.25.2-alpine3.18` and tag `v3`

## ConfigMaps
- from literal - Curl a remote URL and store the result in a file
  - Create a config map `cm-env` with a literal value `REMOTE_URL=http://www.google.com`
  - Create a pod with image `alpine/curl`
  - Add a volume with external hostPath volume to `/var/log/curl/` and mount it to `/tmp/`
  - Execute `curl $REMOTE_URL` and store it to `/tmp/result-curl.txt`
  - check the file in the host

- from directory - Create nginx web server to display files from the config maps
  - Create in a directory the files: `index.html` with the content `Hello world!` and `bye.html` with `Bye world!`
  - Create a config map `cm-dir` with previous directory
  - Create a pod with image `nginx` expose port `80`
  - Add previous config map as volume to `/usr/share/nginx/html/`
  - curl the pod and check the output of both files

- from file - Create nginx web server to display configuration from the config map using
  - a literal value -> configure and print a value using ENV
    - init container prints `env` into `/usr/share/nginx/html/env.html`
  - a literal value -> list of values using config map data
  - directory of files

- Create nginx web server to display configuration from the config map using
  - a literal value -> configure and print a value using ENV
    - init container prints `env` into `/usr/share/nginx/html/env.html`
  - a literal value -> list of values using config map data
  - directory of files

## Secrets


## Services

## Volumes
- Create a pod with a `hostPath` volume to store the nginx access logs (`/var/log/nginx/`) in the host filesystem (`/var/nginx-log`).
  - create some traffic and check the logs
  - delete the pod
  - create a new pod and check if the file is still there
- Replace `hostPath` configured earlier with the newly created `PersistentVolumeClaim`.
  - generate traffic and check the logs
  - what is the Retain Policy of the `PersistentVolume`? What happens if you delete the pod? What happens if you delete the `PersistentVolume`?

## Security Contexts and user privileges

## Service accounts

## Network policies

-----

```bash
for i in {1..5}; do
   kubectl exec --namespace=kube-public curl -- sh -c 'test=`wget -qO- -T 2  http://webapp-service.default.svc.cluster.local:8080/info 2>&1` && echo "$test OK" || echo "Failed"';
done
```
