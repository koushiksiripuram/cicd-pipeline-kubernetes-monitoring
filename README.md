# CI/CD Pipeline with Monitoring for Maven Application

A complete end-to-end DevOps project implementing a CI/CD pipeline with full monitoring stack on AWS EC2 and Kubernetes.

---

## 🏗️ Architecture Overview

```
GitHub → Jenkins → Maven → SonarQube → Trivy → Nexus → Docker → Kubernetes
                                                                      ↓
                                                 Grafana ← Prometheus ← Exporters
```

---

## 🛠️ Tech Stack

| Category | Tools |
|----------|-------|
| CI/CD | Jenkins |
| Build | Maven |
| Code Quality | SonarQube |
| Security Scan | Trivy |
| Artifact Repo | Nexus Repository |
| Containerization | Docker, Docker Hub |
| Orchestration | Kubernetes (Master + 2 Workers) |
| Networking | Flannel CNI |
| Monitoring | Prometheus, Grafana |
| Exporters | Blackbox Exporter, Node Exporter |
| Cloud | AWS EC2 |
| Version Control | GitHub |

---

## 🖥️ Infrastructure

| Server | Purpose | Instance Type |
|--------|---------|---------------|
| Jenkins Server | CI/CD + SonarQube + Nexus | t3.small |
| Kubernetes Master | K8s Control Plane | t3.micro |
| Worker Node 1 | App Workloads | t3.small |
| Worker Node 2 | App Workloads | t3.small |
| Prometheus Server | Monitoring + Grafana | t3.small |

---

## ⚡ Jenkins Pipeline Stages

1. **Tool Install** — Install required tools
2. **Git Checkout** — Clone source code from GitHub
3. **Compile** — Maven compile
4. **Test** — Run unit tests
5. **File System Scan** — Trivy filesystem vulnerability scan
6. **SonarQube Analysis** — Static code analysis
7. **Quality Gate** — SonarQube quality gate check
8. **Build** — Maven package
9. **Publish to Nexus** — Upload artifact to Nexus
10. **Build & Tag Docker Image** — Build Docker image
11. **Docker Image Scan** — Trivy image vulnerability scan
12. **Push Docker Image** — Push to Docker Hub
13. **Deploy to Kubernetes** — Apply K8s manifests
14. **Verify Deployment** — Confirm pods are running

---

## 📊 Monitoring Stack

- **Prometheus** — Metrics collection and storage
- **Grafana** — Visualization and dashboards
- **Blackbox Exporter** — HTTP endpoint monitoring (port 9115)
- **Node Exporter** — System-level metrics (port 9100)

### Prometheus Targets

| Job | Target | Purpose |
|-----|--------|---------|
| prometheus | localhost:9090 | Self monitoring |
| blackbox | App NodePort URL | Endpoint probing |
| node_exporter | Jenkins VM:9100 | System metrics |
| jenkins | Jenkins VM:8080 | Pipeline metrics |

---

## 🚀 Setup Guide

### Prerequisites

- AWS Account with EC2 access
- Minimum 4 EC2 instances (t3.small or above)
- Security group with required ports open

### Required Ports

| Port | Service |
|------|---------|
| 8080 | Jenkins |
| 9000 | SonarQube |
| 8081 | Nexus |
| 9090 | Prometheus |
| 3000 | Grafana |
| 9115 | Blackbox Exporter |
| 9100 | Node Exporter |
| 6443 | Kubernetes API |
| 30000-32767 | Kubernetes NodePorts |
| 4789/UDP | Flannel VXLAN |

### 1. Kubernetes Cluster Setup

```bash
# On Master Node
kubeadm init --pod-network-cidr=10.244.0.0/16

# Install Flannel CNI
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Install Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
```

### 2. Deploy Application

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### 3. Setup Prometheus

```bash
# Start Prometheus
./prometheus --config.file=prometheus.yml

# Start as service
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

### 4. Setup Blackbox Exporter

```bash
./blackbox_exporter --config.file=blackbox.yml
sudo systemctl start blackbox_exporter
sudo systemctl enable blackbox_exporter
```

### 5. Setup Node Exporter

```bash
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

---

## 📁 Project Structure

```
├── Jenkinsfile                  # Jenkins pipeline definition
├── kubernetes/
│   ├── deployment.yaml          # K8s deployment manifest
│   └── service.yaml             # K8s service manifest
├── monitoring/
│   ├── prometheus.yml           # Prometheus configuration
│   └── blackbox.yml             # Blackbox exporter config
├── docker/
│   └── Dockerfile               # Docker image definition
└── README.md
```

---

## 📈 Grafana Dashboards

| Dashboard | Import ID |
|-----------|-----------|
| Node Exporter Full | 1860 |
| Blackbox Exporter | 7587 |
| Jenkins | 9964 |

---

## 🔧 Key Learnings

- Troubleshooting Kubernetes CNI networking issues (Calico BGP → Flannel VXLAN)
- Configuring Prometheus scrape jobs for multiple targets
- Building Grafana dashboards with custom PromQL queries
- Setting up systemd services for monitoring exporters
- End-to-end pipeline automation with Jenkins declarative pipelines

---

## 📬 Contact

Feel free to connect on [LinkedIn](https://linkedin.com) or raise an issue if you have any questions!
