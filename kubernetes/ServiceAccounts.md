# Service accounts

Kubernetes service accounts are used to provide an identity for processes that run in a Pod.

## Get service account information


```bash
kubectl get sa

kubectl get serviceaccounts 
NAME      SECRETS   AGE
default   0         9m4s
dev       0         3m11s
```

## Create service account

Get help:

```bash
kubectl create sa -h
Create a service account with the specified name.

Aliases:
serviceaccount, sa

Examples:
  # Create a new service account named my-service-account
  kubectl create serviceaccount my-service-account

  (...)
```

Create a service account:
```bash
 kubectl create sa dashboard-sa
serviceaccount/dashboard-sa created
```

## Attach token to a service account

```bash
kubectl create token -h
Usage:
  kubectl create token SERVICE_ACCOUNT_NAME [options]


kubectl create token dashboard-sa
eyJhbGciOiJSUzI1NiIsImtpZCI6Ino3bXB...
```


## Attach service account to a pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-dashboard
  namespace: default
spec:
  replicas: 1
  (...)
  template:
    metadata:
      labels:
        name: web-dashboard
    spec:
      serviceAccountName: dashboard-sa
      containers:
```
