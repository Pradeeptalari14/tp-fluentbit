# Fluent Bit Log Processor & Forwarder Studio (tp-fluentbit)

[![SLSA Level 3](https://img.shields.io/badge/SLSA-Level%203-success?style=flat-square&logo=sigstore)](https://slsa.dev)
[![DevSecOps 9+](https://img.shields.io/badge/DevSecOps-9%2B%20Pillars-blue?style=flat-square&logo=kubernetes)](https://talaripradeep.info)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)

> **Production Repository:** [`Pradeeptalari14/tp-fluentbit`](https://github.com/Pradeeptalari14/tp-fluentbit)  
> **Interactive Studio Workspace:** [https://talaripradeep.info/tools/fluent-bit/](https://talaripradeep.info/tools/fluent-bit/)  
> **Visual Flow Canvas:** [https://talaripradeep.info/studios/flow/?studio=fluent-bit](https://talaripradeep.info/studios/flow/?studio=fluent-bit)  
> **Architecture Center:** [https://talaripradeep.info/architecture/](https://talaripradeep.info/architecture/)

---

## 🏛️ Overview
High-performance, lightweight log processor and forwarder for Kubernetes with metadata enrichment and PII masking.

This repository is built following the **9+ DevSecOps / SRE Governance Principles**:
1. **Cosign SLSA L3**: Keyless container and manifest attestation recorded to Rekor.
2. **Semgrep AST Guardrails**: Pre-commit AST rules preventing taint paths and plaintext credentials.
3. **CycloneDX SBOM**: Syft-generated Software Bill of Materials with Grype zero High/Critical CVE gating.
4. **Kyverno Admission Policies**: Enforcing CIS Kubernetes hardening, non-root users, and read-only filesystems.
5. **eBPF In-Kernel Runtime Observability**: Tetragon tracing and live syscall profiling.
6. **GitOps Zero-Drift Synchronization**: ArgoCD/Flux continuous declarative reconciliation.
7. **MELT Golden Signals**: OpenTelemetry traces, Prometheus metrics, and structured logging.
8. **Automated SRE Remediation**: Self-healing circuits and SLO compliance.
9. **Zero-Trust Microsegmentation**: Strict SPIFFE/mTLS and encrypted storage volumes.

---

## 📄 Primary Configuration: `fluent-bit.conf`

```yaml
[SERVICE]
    Flush         1
    Log_Level     info
    Daemon        off
    Parsers_File  parsers.conf
    HTTP_Server   On
    HTTP_Listen   0.0.0.0
    HTTP_Port     2020

[INPUT]
    Name              tail
    Tag               kube.*
    Path              /var/log/containers/*.log
    Parser            docker
    DB                /var/log/flb_kube.db
    Mem_Buf_Limit     50MB
    Skip_Long_Lines   On
    Refresh_Interval  10

[FILTER]
    Name                kubernetes
    Match               kube.*
    Kube_URL            https://kubernetes.default.svc:443
    Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token
    Merge_Log           On
    Keep_Log            Off

[FILTER]
    Name          rewrite_tag
    Match         kube.*
    Rule          $kubernetes['namespace_name'] ^(production)$ prod.$TAG false

[OUTPUT]
    Name            opensearch
    Match           prod.*
    Host            opensearch-cluster.logging.svc
    Port            9200
    Index           production-k8s-logs
    Type            _doc
    HTTP_User       elastic
    HTTP_Passwd     ${LOGGING_SECRET}
    tls             On
    tls.verify      On

```

---

## 🚀 Quickstart & Verification

```bash
# 1. Clone repository
git clone https://github.com/Pradeeptalari14/tp-fluentbit.git
cd tp-fluentbit

# 2. Run verification drill
bash scripts/validate.sh
```

---

## 📜 License
MIT License. Copyright (c) 2026 Talari Pradeep. All rights reserved.
