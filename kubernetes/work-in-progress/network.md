# Network

- If a Pod is not selected by any NetworkPolicies, it is non-isolated. All traffic (incoming and outgoing) will be allowed.
- NetworkPolicies allows to control network access within the cluster network, only allowing the traffic that is needed.
- The empty podSelector will apply the NetworkPolicy to all Pods in the same Namespace as the NetworkPolicy.

NodePort Services are exposed externally by listening on a port on all cluster nodes.


## Create a Service to Expose the Application

The application's Deployment is called hive-io-frontend and exists in the hive Namespace.

The application's web server listens on port 80. Create a Service called hive-io-frontend-svc to expose this application on port 8080. The Service only needs to expose the application within the cluster network.

```
apiVersion: v1
kind: Service
metadata:
  name: hive-io-frontend-svc
  namespace: hive
spec:
  type: ClusterIP
  selector:
    app: hive-io-frontend
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```


## Expose the Application Externally Using an Ingress

Create an Ingress to expose the application.

An Ingress controller is already installed. You can use nginx for the ingressClassName.

Configure the application use hive.io for the domain. The lab server already has an entry in /etc/hosts set up for this domain, so you do not need to make any changes to /etc/hosts.

Remember that the application's Service listens on port 8080.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hive-io-frontend-ingress
  namespace: hive
spec:
  ingressClassName: nginx
  rules:
  - host: hive.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hive-io-frontend-svc
            port:
              number: 8080
```
