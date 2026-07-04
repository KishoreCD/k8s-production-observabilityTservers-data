# k8s-production-observabilityTservers-data
A local-first engineering project replicating a production Prometheus/Grafana infrastructure failure inside Minikube using Terraform.
# Production Observability Pipeline Replication: Root Cause Analysis & Local Isolation

This project is a standalone technical case study designed for DevOps/SRE interview preparation. It focuses on replicating, isolating, and resolving a high-priority production observability issue within a tightly controlled local environment. 

By simulating production-grade container orchestration locally, this repository demonstrates how to bridge the gap between production infrastructure bottlenecks and local debugging topologies.

## 🛠️ The Tech Stack

- Infrastructure as Code (IaC):Terraform (Declarative local state & provider management)
- Container Orchestration:Kubernetes / Minikube (Docker driver runtime)
- Observability Framework:Prometheus Operator CRDs & Grafana Dashboards
## 📐 Root Cause Analysis & Architectural Gotchas

### 1. The Local Image Context Trap
The Production Reality: Production nodes fetch images seamlessly from distributed enterprise registries (e.g., AWS ECR, Google GCR).
The Local Bottleneck: Running `minikube docker-env` shifts the shell context directly *inside* the cluster engine daemon. Attempting to build or reference host-level images there throws a silent `reference does not exist` error.

The Resolution: Keep the host environment isolated. Build images natively on the host machine and pass the built container assets cleanly into the cluster using native file caching:

  minikube image load <image_name>:<tag>
