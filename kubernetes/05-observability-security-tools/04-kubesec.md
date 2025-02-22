# Kubesec

KubeSec is useful for security risk assessment of Kubernetes workloads. Unlike [KubeLinter](02-kubelinter.md), which enforces best practices, KubeSec assigns a security score and provides actionable recommendations based on its analysis.


1. Security Risk Scoring
- KubeSec assigns a risk score to Kubernetes manifests.
- A higher score = more secure configuration.
- This helps to prioritize security fixes based on severity.
  
2. Identifies Common Security Issues like
- Running as root (`runAsNonRoot: false`)
- Privileged containers (`privileged: true`)
- Missing read-only root filesystem (`readOnlyRootFilesystem: false`)
- Excessive capabilities (e.g., `NET_ADMIN`)
- Provides explanations and recommendations for each issue.

## Installation

Download the [latest release](https://github.com/controlplaneio/kubesec/releases) of KubeSec from the GitHub repository:

```bash
wget https://github.com/controlplaneio/kubesec/releases/download/v2.14.2/kubesec_linux_amd64.tar.gz

tar -xvf  kubesec_linux_amd64.tar.gz

mv kubesec /usr/bin/
```

## Scanning Kubernetes Manifests

Scan a Kubernetes manifest file using the following command:

```bash
$ kubesec scan pod.yaml 
[
  {
    "object": "Pod/node.default",
    "valid": true,
    "fileName": "pod.yaml",
    "message": "Failed with a score of -27 points",
    "score": -27,
    "scoring": {
      "critical": [
        {
          "id": "Privileged",
          "selector": "containers[] .securityContext .privileged == true",
          "reason": "Privileged containers can allow almost completely unrestricted host access",
          "points": -30
        }
      ],
      "passed": [
        {
          "id": "ServiceAccountName",
          "selector": ".spec .serviceAccountName",
          "reason": "Service accounts restrict Kubernetes API access and should be configured with least privilege",
          "points": 3
        }
      ],
    ...
```

The output provides a detailed analysis of the security risks in the manifest file, including the risk score, reasons for the score, and recommendations for remediation.

In the previous example, the `Privileged` issue was identified, which resulted in a score of -30 points. The recommendation is to avoid using privileged containers to restrict host access.

```yaml
    securityContext:
      privileged: true
      readOnlyRootFilesystem: false
```

Change the `privileged` field to `false` will address the issue.


## 📚 References

- [KubeSec](https://kubesec.io/)