# Trivy

Trivy is an open-source vulnerability scanner that detects security issues in:

- Container images (Docker, Kubernetes)
- Filesystem (host, infrastructure)
- SBOM files (SPDX, CycloneDX)
- Git repositories
- Kubernetes clusters

## Installation

Refer to the [official documentation](https://aquasecurity.github.io/trivy/v0.19.2/installation/install/).

```bash
#Add the trivy-repo
sudo apt-get install wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" |  sudo tee -a /etc/apt/sources.list.d/trivy.list

#Update Repo and Install trivy
sudo apt-get update
sudo apt-get install trivy
```

## Scanning Container Images

To scan a Docker image using Trivy, use the following command:

```bash
$  trivy image --severity HIGH --output /root/python.json --format json public.ecr.aws/docker/library/python:3.13-bullseye 
INFO    [vuln] Vulnerability scanning is enabled
INFO    [secret] Secret scanning is enabled
INFO    Detected OS     family="debian" version="11.11"
INFO    [debian] Detecting vulnerabilities...   os_version="11" pkg_num=426
INFO    Number of language-specific files       num=1
INFO    [python-pkg] Detecting vulnerabilities...

$  head python.json 
{
  "SchemaVersion": 2,
  "ArtifactName": "public.ecr.aws/docker/library/python:3.13-bullseye",
  "ArtifactType": "container_image",
  "Metadata": {
    "OS": {
      "Family": "debian",
      "Name": "11.11"
    },
 ...
```

In the example above, Trivy scans the `public.ecr.aws/docker/library/python:3.13-bullseye` image for vulnerabilities with a severity level of `HIGH` and outputs the results in JSON format to the `python.json` file.


