# 1. Securing & Hardening Kubernetes Deployments

- [1. Securing & Hardening Kubernetes Deployments](#1-securing--hardening-kubernetes-deployments)
  - [Securing Supply Chains](#securing-supply-chains)
    - [Scanning for CVE's](#scanning-for-cves)

## Securing Supply Chains

Effective Supply chain security is becoming more critical due to the effects of supply chain attacks in the wild. The following whitepaper coveres supply chain security in-depth:
https://github.com/cncf/tag-security/blob/main/supply-chain-security/supply-chain-security-paper/CNCF_SSCP_v1.pdf

### Scanning for CVE's

It is recommended to scan container images and application code for vulnerabilities before running code in production. Tools can scan dependancies and code for publicly known vulnerabilities and should be considered as part of a minimum viable security measure to impelenting container security. CVE scanning tools can detect code vulnerabilities:
- in container images
- in filesystems & dependancies
- in git repositories  

Tools
- Trivy - https://github.com/aquasecurity/trivy
- Clair - https://github.com/quay/clair
- Anchore/Grype - https://anchore.com/opensource/
- Dagada - https://github.com/eliasgranderubio/dagda/


intrusion detections - detect new shells spawned in containers:
- tracee
- falco