---
layout: default
title: "Κεφάλαιο D8: Docker Security Hardening με Ansible"
sort: 8
---

# D8: Docker Security Hardening με Ansible 🔒

## Τι θα καλύψουμε

```
D8.1 Security Concepts για Docker
     ├── Attack surface
     └── Defense in depth

D8.2 Container Runtime Security
     ├── no-new-privileges
     ├── read_only rootfs
     ├── Non-root user
     └── Capabilities

D8.3 Secrets Management
     ├── Αποφυγή env var secrets
     ├── Docker secrets (Swarm)
     └── Ansible Vault + volume pattern

D8.4 Image Security
     ├── Minimal base images
     ├── Non-root Dockerfile
     └── Image scanning με Trivy

D8.5 Network Security
     ├── Internal networks
     ├── Publish μόνο απαραίτητα ports
     └── Firewall integration

D8.6 Πρακτικό παράδειγμα
     └── Hardened production stack
```

## Υποπαράγραφοι
- [D8.1 Security Concepts για Docker 🛡️](./8.1-docker-security-concepts.md)
- [D8.2 Container Runtime Security ⚙️](./8.2-container-runtime-security.md)
- [D8.3 Secrets Management 🔐](./8.3-secrets-management.md)
- [D8.4 Image Security 🖼️](./8.4-image-security.md)
- [D8.5 Network Security 🌐](./8.5-network-security.md)
- [D8.6 Πρακτικό παράδειγμα — Hardened Production Stack 🚀](./8.6-hardened-stack-example.md)
