# Kubernetes Learning Notes  

This repository contains structured notes and resources for learning and practicing Kubernetes. The content is organized into different topics to facilitate learning and reference.  

## 📂 Folder Structure  

### 01 - Architecture  
Fundamental concepts about Kubernetes architecture and how to interact with the cluster.  

- 📄 [01-architecture.md](01-architecture/01-architecture.md) – Overview of Kubernetes architecture.  
- 📄 [02-kubectl.md](01-architecture/02-kubectl.md) – Using `kubectl` to interact with the cluster.  
- 📄 [03-nodes.md](01-architecture/03-nodes.md) – Nodes and their role in Kubernetes.  

### 02 - Core Concepts  
Key components and objects in Kubernetes.  

- 📄 [01-namespaces.md](02-core-concepts/01-namespaces.md) – Organizing workloads using namespaces.  
- 📄 [02-pods.md](02-core-concepts/02-pods.md) – The smallest deployable unit in Kubernetes.  
- 📄 [03-jobs.md](02-core-concepts/03-jobs.md) – Running batch jobs.  
- 📄 [04-assign-pods-nodes.md](02-core-concepts/04-assign-pods-nodes.md) – Controlling pod placement.  
- 📄 [05-deployments.md](02-core-concepts/05-deployments.md) – Managing applications with deployments.  
- 📄 [06-updating-deployment.md](02-core-concepts/06-updating-deployment.md) – Updating running applications.  
- 📄 [07-replica-sets.md](02-core-concepts/07-replica-sets.md) – Scaling applications using ReplicaSets.  
- 📄 [08-services.md](02-core-concepts/08-services.md) – Exposing applications using services.  
- 📄 [09-volumes.md](02-core-concepts/09-volumes.md) – Managing storage for pods.  
- 📄 [10-configmaps.md](02-core-concepts/10-configmaps.md) – Configuring applications with ConfigMaps.  
- 📄 [11-secrets.md](02-core-concepts/11-secrets.md) – Handling sensitive data.  
- 📄 [12-persistent-volumes.md](02-core-concepts/12-persistent-volumes.md) – Persistent storage in Kubernetes.  
- 📄 [13-resource-quotas.md](02-core-concepts/13-resource-quotas.md) – Managing resource limits in a namespace.
- 📄 [14-labels.md](02-core-concepts/14-labels.md) – Organizing resources with labels.

### 03 - Networking  
How Kubernetes handles networking between pods, services, and external systems.  

- 📄 [01-networking.md](03-networking/01-networking.md) – Overview of Kubernetes networking.  
- 📄 [02-debug-network.md](03-networking/02-debug-network.md) – Debugging network issues in Kubernetes.  
- 📄 [03-coredns.md](03-networking/03-coredns.md) – DNS in Kubernetes using CoreDNS.  

### 04 - Security  
Security principles and practices in Kubernetes.  

- 📄 [01-security-contexts.md](04-security/01-security-contexts.md) – Security contexts and how they affect pods.  
- 📄 [02-service-accounts-rbac.md](04-security/02-service-accounts-rbac.md) – Managing permissions with ServiceAccounts and RBAC.  
- 📄 [03-admission-controller.md](04-security/03-admission-controller.md) – Controlling requests to the API server.  
- 📄 [04-pod-security-standards.md](04-security/04-pod-security-standards.md) – Pod security standards.  
- 📄 [05-allowed-registries.md](04-security/05-allowed-registries.md) – Restricting image sources using allowed registries.  
- 📄 [06-audit-logs.md](04-security/06-audit-logs.md) – Tracking cluster activity with audit logs.  
- 📄 [07-encryption.md](04-security/07-encryption.md) – Encryption for data and pod communication.  

### 05 - Observability & Security Tools  
Monitoring, security scanning, and vulnerability assessment tools.  

- 📄 [01-falco.md](05-observability-security-tools/01-falco.md) – Runtime security monitoring with Falco.  
- 📄 [02-kubelinter.md](05-observability-security-tools/02-kubelinter.md) – Linting Kubernetes configurations.  
- 📄 [03-kubesec.md](05-observability-security-tools/03-kubesec.md) – Security analysis with Kubesec.  
- 📄 [04-SBOM-syft-grype.md](05-observability-security-tools/04-SBOM-syft-grype.md) – Software Bill of Materials (SBOM) with Syft and Grype.  
- 📄 [05-trivy.md](05-observability-security-tools/05-trivy.md) – Scanning container images for vulnerabilities.  

### 06 - Setup  
Setting up and managing Kubernetes clusters.  

- 📄 [01-environment-setup.md](06-setup/01-environment-setup.md) – Setting up a Kubernetes environment.  
- 📄 [02-update-k8s.md](06-setup/02-update-k8s.md) – Updating Kubernetes clusters.  
- 📄 [03-docker.md](06-setup/03-docker.md) – Using Docker with Kubernetes.  

### Tooling  
Tools for managing Kubernetes deployments.  

- 📄 [helm.md](tooling/helm.md) – Helm package manager for Kubernetes.  


---

## 📌 Notes  

- These files serve as a structured study guide for Kubernetes concepts.  
- The **work-in-progress** section contains drafts that may be updated over time.  

🚀 Happy Learning!  
