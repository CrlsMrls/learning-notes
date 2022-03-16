# Services

Pods aren't meant to be persistent. They can be stopped or started for many reasons - like failed liveness or readiness checks - and this leads to a problem:

What happens if you want to communicate with a set of Pods? When they get restarted they might have a different IP address.

That's where Services come in. _Services provide stable endpoints for Pods_.

Services use labels to determine what Pods they operate on. If Pods have the correct labels, they are automatically picked up and exposed by our services.

The level of access a service provides to a set of pods depends on the Service's type. Currently there are three types:

- **ClusterIP (internal)**: the default type means that this Service is only visible inside of the cluster,
- **NodePort**: gives each node in the cluster an externally accessible IP and
- **LoadBalancer**: adds a load balancer from the cloud provider which forwards traffic from the service to Nodes within it.

http://kubernetes.io/docs/user-guide/services/
