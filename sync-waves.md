# ArgoCD Sync Wave Strategy

This document defines the **approved ArgoCD sync-wave ordering** used for
production Kubernetes clusters.

Sync waves enforce **hard dependency ordering** during cluster bootstrap
and reconciliation. Resources in a lower wave **must fully converge**
before higher waves are applied.

---

## Final Approved Sync Order

| Sync Wave | Layer | Description |
|:---------:|-------|-------------|
| **-30** | ArgoCD primitives | AppProjects, ArgoCD RBAC, Argo-level guardrails |
| **-20** | Root orchestration | Root App-of-Apps (environment bootstrap) |
| **-10** | Namespaces | Platform and application namespaces |
| **-5** | CRDs | cert-manager, external-secrets, gatekeeper CRDs |
| **0** | Controllers | cert-manager, external-secrets, gatekeeper controllers, metrics-server |
| **10** | Platform services | Monitoring, logging, observability stack |
| **20** | Security enforcement | Gatekeeper constraints (audit → enforce) |
| **30** | Ingress & networking | Ingress controllers, DNS, load balancers |
| **40** | ApplicationSets | Business Apps |
| **50** | Stateful workloads | Databases, message queues, caches |
| **60** | Backend services | APIs, background workers, internal services |
| **70** | Frontend services | UI applications, public endpoints |

---

## Sync Wave Design Principles

- Sync waves exist to manage **hard dependencies**, not architectural layers
- CRDs are always applied **before** their controllers
- Security enforcement is **staged**, never during initial bootstrap
- Platform services must stabilize before workloads are deployed
- Stateful services are deployed before dependent applications
- Frontend applications are deployed last to avoid partial availability
- Fewer, well-defined waves result in **faster reconciliation and recovery**

---

## Usage Example

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "50"
