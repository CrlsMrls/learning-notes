# Configuring pods

## Environment variables

Simply add `env` in image section.

```bash
apiVersion: v1
kind: Pod
metadata:
  labels:
    tier: frontend
  name: front-ui
spec:
  containers:
  - image: nginx
    name: front-ui
    ports:
     -containerPort: 80
    env:
     - name: BASE_PATH
       value: web
```
