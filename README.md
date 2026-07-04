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

he Resolution: Force the Prometheus Operator to search across alternative namespaces by toggling podMonitorSelectorNilUsesHelmValues to false in your declarative configuration values:

YAML
prometheus:
  prometheusSpec:
    podMonitorSelectorNilUsesHelmValues: false

3. Named Port Requirements for PodMonitors
The Production Reality: Strict type-matching inside Custom Resource Definitions (CRDs).

The Local Bottleneck: Pointing a structural PodMonitor CRD to a raw integer port string (e.g., "8000") results in a silent "No data" placeholder in Prometheus.

The Resolution: Explicitly declare a name block on your container port specification in your Kubernetes deployment configuration, and match that exact string token inside your PodMonitor spec:

YAML
# Deployment Spec Port Definition
ports:
  - name: http-metrics
    containerPort: 8000
Deployment & Verification Lifecycle
Prerequisites
Minikube

Terraform

Docker

Execution
Initialize the local cluster:

Bash
minikube start --driver=docker


Build and cache the application asset:

Bash
docker build -t app:latest .
minikube image load app:latest

Orchestrate infrastructure via IaC:

Bash
terraform init
terraform apply -auto-approve

Expose metrics interface to host machine:

Bash
kubectl port-forward deployment/kube-prometheus-stack-grafana 3000:3000

ASSET:
<img width="1868" height="958" alt="Screenshot 2026-07-04 042819" src="https://github.com/user-attachments/assets/a3383d8a-8137-41ad-8e52-7e3875689e9c" />
<img width="1232" height="813" alt="Screenshot 2026-07-04 051557" src="https://github.com/user-attachments/assets/6b4adc47-d333-45eb-91e9-db573a93fe58" />
