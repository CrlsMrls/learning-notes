
Multi-container patterns help the main container, these are the most common patterns: 
- A sidecar container performs some task that helps the main container.
- An ambassador container proxies network traffic to and/or from the main container.
- An adapter container transforms the main container's output.

Multi-container pods allow tightly coupled containers to share resources. All containers in the Pod:
- can access the shared volumes, allowing those containers to share data. 
- share the network namespace, including the IP address and network ports. Inside a Pod, the containers can communicate with one another using `localhost`. 

https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/



# Jobs

The `restartPolicy` for a Job or CronJob pod must be `OnFailure` or `Never`.

Use `activeDeadlineSeconds` in the Job spec to determinate the Job duration it it runs too long