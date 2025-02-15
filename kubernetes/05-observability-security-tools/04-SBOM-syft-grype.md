# SBOM (Software Bill of Materials)  

SBOM is a list of components in a software system. It is a detailed inventory of all the components that make up a software product. This is useful for tracking and managing software components, dependencies, and licenses. 


## Formats

The main two formats for SBOM are SPDX and CycloneDX. CycloneDX is a newer format that is gaining popularity as it is focused on security.

### SPDX

This format is best for legal and open-source tracking. 

- Focuses on license compliance and legal obligations.
- Helps track open-source components and their licenses.
- Useful for legal and regulatory requirements.

### CycloneDX

This format is best for security and DevSecOps.

Purpose:
- Focuses on application security and vulnerability management.
- Provides dependency graphs for software components.
- Helps in detecting vulnerabilities (CVEs) and risks in software supply chains.
  
## Syft

Syft is an open-source tool that can be used to scan container images and generate SBOMs in various formats.

### Installation

Install Syft using the following command:

```bash
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
```

### Scanning

To scan a Docker image and generate an SBOM, use the following command:

```bash
$ syft scan docker.io/nginx:latest -o spdx > nginx.sbom
 ✔ Loaded image                            index.docker.io/library/nginx:latest
 ✔ Parsed image                    sha256:39286ab8a5e14aeaf5fdd6e2fac76e0c8d31a  
 ✔ Cataloged contents              edf555d07d2ddbe6b616d90ee444e2faec1a219310c9  
   ├── ✔ Packages                        [150 packages]  
   ├── ✔ File digests                    [3,710 files]  
   ├── ✔ File metadata                   [3,710 locations]  
   └── ✔ Executables                     [842 executables]  
```

To scan the same image and generate an SBOM in CycloneDX format, use the following command:

```bash
$ syft scan docker.io/nginx:latest -o cyclonedx-json > nginx-cyclonedx.json
 ✔ Loaded image                            index.docker.io/library/nginx:latest
 ✔ Parsed image                    sha256:39286ab8a5e14aeaf5fdd6e2fac76e0c8d31a  
 ✔ Cataloged contents              edf555d07d2ddbe6b616d90ee444e2faec1a219310c9  
   ├── ✔ Packages                        [150 packages]  
   ├── ✔ File digests                    [3,710 files]  
   ├── ✔ File metadata                   [3,710 locations]
   └── ✔ Executables                     [842 executables]
```

The previous command will generate an SPDX format file. Other possible formats include: cyclonedx-json, cyclonedx-xml, github-json, spdx-json, spdx-tag-value, syft-json, syft-table, syft-text and template


## Grype

Grype is an open-source tool that scans SBOMs for vulnerabilities. It works alongside Syft, which generates the SBOM. It can be used to generate SBOMs in CycloneDX format.

Grype requires SPDX JSON, CycloneDX JSON, or Syft JSON formats.

### Installation

Install Grype using the following command:

```bash
curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin
```

### Scanning

To scan a Docker image and generate an SBOM, use the following command:

```bash
grype sbom:nginx.sbom -o json > nginx.json
 ✔ Scanned for vulnerabilities     [169 vulnerability matches]  
   ├── by severity: 6 critical, 17 high, 45 medium, 12 low, 85 negligible (4 unkn
   └── by status:   22 fixed, 147 not-fixed, 0 ignored 
```

Similarly, you can generate an SBOM in CycloneDX format using the following command:

```bash
 grype nginx-cyclonedx.json -o json > nginx-vulnerabilities.json
 ✔ Scanned for vulnerabilities     [169 vulnerability matches]  
   ├── by severity: 6 critical, 17 high, 45 medium, 12 low, 85 negligible (4 unkn
   └── by status:   22 fixed, 147 not-fixed, 0 ignored 
```

