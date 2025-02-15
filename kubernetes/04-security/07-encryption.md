# Encryption in Kubernetes

Encryption is crucial for securing data in transit and at rest within a Kubernetes cluster. Kubernetes does not encrypt pod-to-pod communication by default, but various mechanisms can be used to enforce encryption at different layers.

## Encryption in Pod-to-Pod Communication

By default, pod-to-pod communication within a Kubernetes cluster occurs over an internal network that may not be encrypted. To secure pod-to-pod traffic, you can use Cilium WireGuard, mTLS, or IPsec.

### Cilium WireGuard Encryption
Cilium provides transparent encryption using WireGuard, a modern and efficient VPN protocol that encrypts network traffic at the kernel level. When enabled, all pod-to-pod traffic within a cluster is encrypted using WireGuard.

To verify that pod-to-pod communication is encrypted, you can inspect network packets using tcpdump.

### Verifying Encryption with tcpdump
To capture and inspect traffic on the WireGuard interface used by Cilium, run the following command:

```bash
tcpdump -n -i cilium_wg0 -X
```

This will display packet contents in hexadecimal and ASCII format, allowing you to verify that the data is encrypted.

If encryption is enabled, captured traffic will appear unreadable, showing random-looking bytes instead of plaintext data.

If encryption is disabled, you may see readable HTTP or other protocol contents.

## Encrypting Secrets at Rest

Kubernetes stores sensitive data, such as Secrets, in etcd. By default, data stored in etcd is not encrypted at rest, which poses a security risk.

To enable encryption of Secrets at rest, you must configure an EncryptionConfiguration in the Kubernetes API server.

### Enabling etcd Encryption
Create an EncryptionConfiguration file, e.g., `/etc/kubernetes/encryption-config.yaml`:
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <base64-encoded-key>
      - identity: {}
```

Replace <base64-encoded-key> with a 256-bit Base64-encoded encryption key.

Update the API server to use the encryption configuration:

```yaml
- --encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

Restart the API server.

Rotate Encryption Keys: If necessary, create a new key, update the configuration, and re-encrypt Secrets.

### Verify Encryption

Extract raw etcd data to confirm that stored Secrets are encrypted.

```bash
ETCDCTL_API=3 etcdctl get /registry/secrets/default/mysecret --hex
```

Encrypted secrets should appear as unreadable data.