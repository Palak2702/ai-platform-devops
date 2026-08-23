# AI Platform DevOps — Kubernetes, Helm, Monitoring & CI/CD

A production-oriented DevOps project demonstrating how to containerize, deploy, operate, monitor, secure, and continuously deliver an AI platform based on **Ollama + Open WebUI**.

The application was deployed locally on a **Kind Kubernetes cluster** and integrated with **Helm, persistent storage, ConfigMaps, Secrets, health probes, Prometheus, Grafana, alerting, GitHub Actions CI/CD, and a self-hosted GitHub Actions runner**.

> **AWS/EKS production deployment:** The project architecture is designed to be portable to AWS EKS. The current implementation in this repository is the validated local/Kind environment; an EKS/Terraform production deployment is the planned next phase and should not be represented as already completed.

---

## 1. Project Objective

The objective was not to develop an AI model. The objective was to demonstrate **DevOps ownership of an AI workload**:

- Containerized AI services
- Kubernetes orchestration
- Persistent model/application data
- Configuration and secret management
- Application health checks
- Helm-based packaging and deployment
- Kubernetes monitoring and alerting
- CI validation and security scanning
- CD deployment through a self-hosted runner
- Operational troubleshooting and recovery

This project is designed to demonstrate skills relevant to **DevOps Engineer / Cloud DevOps Engineer / Kubernetes Engineer / Platform Engineer** roles.

---

## 2. Architecture

```text
                         ┌──────────────────────┐
                         │       GitHub         │
                         │  Source Repository   │
                         └──────────┬───────────┘
                                    │
                              git push / PR
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │   GitHub Actions     │
                         │         CI           │
                         │                      │
                         │ • YAML validation    │
                         │ • Helm lint/template │
                         │ • Security scanning  │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ Self-hosted Runner   │
                         │      Windows         │
                         └──────────┬───────────┘
                                    │
                                    ▼
                              Helm upgrade
                                    │
                                    ▼
                    ┌──────────────────────────────┐
                    │       Kind Kubernetes        │
                    │                              │
                    │  ┌──────────┐ ┌───────────┐ │
                    │  │ Ollama   │ │ Open WebUI│ │
                    │  │ 11434    │ │   8080    │ │
                    │  └────┬─────┘ └─────┬─────┘ │
                    │       │              │       │
                    │      PVC            PVC      │
                    │       │              │       │
                    │       └──────┬───────┘       │
                    │              │               │
                    │       ConfigMap/Secret       │
                    └──────────────┬───────────────┘
                                   │
                         ┌─────────┴─────────┐
                         │                   │
                    Prometheus            Grafana
                         │                   │
                         └─────────┬─────────┘
                                   │
                         Dashboards + Alerts
```

---

## 3. Technology Stack

| Category | Technology |
|---|---|
| AI runtime | Ollama |
| AI Web UI | Open WebUI |
| Containers | Docker |
| Local Kubernetes | Kind |
| Kubernetes packaging | Helm |
| Storage | Kubernetes PVC / local-path storage |
| Configuration | ConfigMap |
| Secrets | Kubernetes Secret |
| Health | Liveness / Readiness probes |
| Monitoring | Prometheus |
| Visualization | Grafana |
| Alerting | Prometheus / Grafana |
| CI/CD | GitHub Actions |
| Runner | GitHub Actions self-hosted runner |
| Security | Container/image security scanning in CI |
| Source control | Git / GitHub |
| Planned cloud target | AWS EKS |
| Planned IaC phase | Terraform |

---

## 4. Repository Structure

```text
ai-platform-devops/
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
│
├── docker/
│   └── compose.yaml
│
├── helm/
│   └── ai-platform/
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── charts/
│       └── templates/
│           ├── _helpers.tpl
│           ├── ollama/
│           │   ├── deployment.yaml
│           │   ├── pvc.yaml
│           │   └── service.yaml
│           └── open-webui/
│               ├── configmap.yaml
│               ├── deployment.yaml
│               ├── pvc.yaml
│               ├── secret.yaml
│               └── service.yaml
│
├── kubernetes/
│   ├── namespace.yaml
│   ├── ollama/
│   │   ├── deployment.yaml
│   │   ├── pvc.yaml
│   │   └── service.yaml
│   └── open-webui/
│       ├── configmap.yaml
│       ├── deployment.yaml
│       ├── pvc.yaml
│       ├── secret.yaml
│       └── service.yaml
│
├── docker/
│   └── compose.yaml
│
├── kind-config.yaml
├── helm-rendered.yaml
└── README.md
```

---

## 5. Kubernetes Components

### Namespace

The application is isolated in:

```text
ai-platform
```

### Ollama

- Deployment
- ClusterIP Service
- PersistentVolumeClaim
- Persistent model storage
- Resource requests/limits
- HTTP port `11434`

### Open WebUI

- Deployment
- NodePort Service
- PersistentVolumeClaim
- ConfigMap
- Secret
- Resource requests/limits
- Liveness probe
- Readiness probe
- HTTP port `8080`

### Storage

Persistent storage prevents application/model data from being tied only to the lifetime of an individual pod.

Validated PVCs included:

```text
ai-platform-ollama-pvc       10Gi
ai-platform-open-webui-pvc    5Gi
```

---

## 6. Configuration & Secret Management

### ConfigMap

Open WebUI receives the Ollama endpoint through a ConfigMap.

Example concept:

```text
OLLAMA_BASE_URL
```

### Secret

Sensitive WebUI configuration is stored using a Kubernetes Secret instead of hard-coding it directly into the Deployment.

The repository should never contain real credentials or production secrets.

---

## 7. Health Probes

The Open WebUI deployment uses:

- Readiness probe
- Liveness probe

Example endpoint:

```text
/health
```

The probes allow Kubernetes to determine whether the application is ready to receive traffic and whether a container needs to be restarted.

---

## 8. Helm

The Kubernetes manifests were converted into a reusable Helm chart.

Useful commands:

```powershell
helm lint helm/ai-platform
helm template ai-platform helm/ai-platform
helm template ai-platform helm/ai-platform > helm-rendered.yaml
```

Install:

```powershell
helm install ai-platform helm/ai-platform -n ai-platform
```

Upgrade:

```powershell
helm upgrade --install ai-platform helm/ai-platform -n ai-platform
```

Verify:

```powershell
helm status ai-platform -n ai-platform
helm list -n ai-platform
```

The chart was successfully linted:

```text
1 chart(s) linted, 0 chart(s) failed
```

---

## 9. Monitoring

The project includes a Prometheus/Grafana monitoring stack installed in the `monitoring` namespace.

Validated components include:

- Prometheus
- Alertmanager
- Grafana
- kube-state-metrics
- Prometheus node exporter
- kube-prometheus-stack

Example verification:

```powershell
kubectl get pods -n monitoring
```

Expected components include:

```text
Alertmanager
Grafana
Prometheus Operator
kube-state-metrics
node-exporter
Prometheus
```

Grafana was used for Kubernetes/application observability and dashboard visualization.

---

## 10. Alerting

Prometheus/Grafana alert rules were created and tested.

The goal is to demonstrate operational monitoring rather than simply displaying dashboards.

Examples of useful production-style alert categories:

- Pod unavailable
- Deployment replicas unavailable
- High CPU
- High memory
- Node not ready
- Persistent volume/storage problems

---

## 11. CI Pipeline

GitHub Actions CI validates the repository before deployment.

The CI stage includes Kubernetes/Helm validation and security scanning.

Typical validation commands include:

```powershell
kubectl apply --dry-run=client -f kubernetes/
helm lint helm/ai-platform
helm template ai-platform helm/ai-platform
```

The pipeline also includes a container/image security scanning step.

---

## 12. CD Pipeline

Deployment is performed through a **GitHub Actions self-hosted runner** connected to the local Kind environment.

The validated flow is:

```text
GitHub
   ↓
GitHub Actions
   ↓
Self-hosted runner
   ↓
kubectl / Helm
   ↓
Kind Kubernetes
   ↓
Ollama + Open WebUI
```

Example deployment command:

```powershell
helm upgrade --install ai-platform helm/ai-platform `
  --namespace ai-platform `
  --create-namespace
```

Deployment verification:

```powershell
kubectl get pods -n ai-platform
kubectl get svc -n ai-platform
helm status ai-platform -n ai-platform
```

---

## 13. Accessing Open WebUI

For the local Kind environment, the reliable development method used during validation was port forwarding:

```powershell
kubectl port-forward svc/ai-platform-open-webui 3000:80 -n ai-platform
```

Then open:

```text
http://localhost:3000
```

Port forwarding was used because NodePort access through `localhost:31311` is not guaranteed to work with the way the Kind cluster is exposed on the host.

---

## 14. Troubleshooting & Recovery

One important operational issue occurred when the Kind control-plane node became `NotReady`.

Symptoms included:

```text
ai-platform-control-plane   NotReady
```

and:

```text
node.kubernetes.io/unreachable:NoSchedule
```

Pods were stuck in:

```text
Pending
```

The node reported:

```text
Kubelet stopped posting node status.
```

### Recovery approach

The Kind control-plane container was restarted:

```powershell
docker restart ai-platform-control-plane
```

After the Docker/Kind environment recovered, Kubernetes returned to:

```text
ai-platform-control-plane   Ready
```

The application pods then returned to:

```text
1/1 Running
```

### Important lesson

The issue was not an application deployment failure. The Kubernetes node itself had become unreachable because the local Docker/WSL environment had restarted or stopped.

This demonstrated practical troubleshooting across:

```text
Host
 ↓
Docker
 ↓
Kind
 ↓
Kubernetes control plane
 ↓
Kubelet
 ↓
Scheduler
 ↓
Application pods
```

---

## 15. Common Command Errors Encountered

### `git` repository error

Initial commands failed because Git had not yet been initialized:

```text
fatal: not a git repository
```

The repository was subsequently initialized and connected to GitHub.

---

### Helm directory error

The first chart creation attempt failed because the parent `helm` directory did not exist:

```text
Error: GetFileAttributesEx ...\helm:
The system cannot find the file specified.
```

After creating the required directory structure, the chart was generated successfully.

---

### Incorrect kubectl logs syntax

An incorrect command was initially used:

```powershell
kubectl get logs <pod>
```

Correct syntax:

```powershell
kubectl logs <pod> -n ai-platform
```

---

### Node unreachable scheduling error

Pending Open WebUI pods showed:

```text
0/1 nodes are available:
1 node(s) had untolerated taint
{node.kubernetes.io/unreachable:}
```

This was traced back to the Kind node being `NotReady`.

---

### kubectl localhost:8080 error

When the Kubernetes context/configuration was unavailable, `kubectl` attempted:

```text
http://localhost:8080
```

and returned:

```text
The connection to the server localhost:8080 was refused
```

The issue was related to Kubernetes context/control-plane availability rather than the application itself.

---

## 16. Local Deployment Flow

### Start Docker Desktop / Docker engine

Verify:

```powershell
docker version
docker ps
```

### Verify Kind

```powershell
kind get clusters
kubectl get nodes
```

### Verify Kubernetes

```powershell
kubectl get pods -A
```

### Verify application

```powershell
kubectl get pods -n ai-platform
kubectl get svc -n ai-platform
```

### Verify Helm

```powershell
helm list -n ai-platform
helm status ai-platform -n ai-platform
```

### Access WebUI

```powershell
kubectl port-forward svc/ai-platform-open-webui 3000:80 -n ai-platform
```

Open:

```text
http://localhost:3000
```

---

## 17. Production / AWS Roadmap

The current repository validates the application and DevOps workflow locally on Kind.

The next production phase is planned as:

```text
Terraform
   ↓
AWS VPC
   ↓
EKS
   ↓
ECR
   ↓
Helm
   ↓
AI Platform
   ↓
Prometheus + Grafana
   ↓
GitHub Actions
```

Planned AWS improvements include:

- Amazon EKS
- Terraform infrastructure as code
- AWS VPC/subnets/security groups
- ECR image registry
- IAM / IRSA
- EKS node groups
- AWS Load Balancer
- Kubernetes Secrets / external secret management
- CloudWatch integration where appropriate
- Production-grade storage
- GitHub Actions deployment to EKS

**These AWS components are a planned production phase and are not claimed as completed in the current repository.**

---

## 18. Skills Demonstrated

### Kubernetes

- Pods
- Deployments
- Services
- Namespaces
- ConfigMaps
- Secrets
- PVCs
- Resource requests/limits
- Health probes
- Scheduling troubleshooting
- Node troubleshooting

### Helm

- Chart creation
- Chart structure
- Templates
- Values
- Linting
- Rendering
- Install
- Upgrade
- Release management

### CI/CD

- GitHub Actions
- Self-hosted runners
- CI validation
- Security scanning
- Kubernetes deployment automation
- Helm-based CD

### Observability

- Prometheus
- Grafana
- Alertmanager
- kube-state-metrics
- Node exporter
- PromQL
- Alert rules

### DevOps

- Git
- GitHub
- Docker
- Kubernetes
- Helm
- CI/CD
- Monitoring
- Troubleshooting
- Infrastructure automation

---

## 19. Resume Keywords

```text
AWS
EKS
Kubernetes
Docker
Helm
GitHub Actions
CI/CD
Prometheus
Grafana
Alertmanager
Kubernetes Monitoring
Observability
Container Security
Secrets Management
ConfigMaps
Persistent Volumes
PVC
Health Probes
Self-hosted GitHub Actions Runner
Infrastructure as Code
Terraform
Linux
Git
Containerization
DevOps
Cloud DevOps
Platform Engineering
```

> Only claim AWS EKS and Terraform as hands-on experience on your resume after completing and validating that phase.

---

## 20. Future Enhancements

- Deploy to AWS EKS
- Provision AWS infrastructure using Terraform
- Push images to Amazon ECR
- Add production ingress/load balancing
- Add IAM/IRSA
- Add centralized logging
- Add GitHub environment approvals
- Add automated rollback strategy
- Add separate development/staging/production environments

---

## 21. Project Outcome

This project demonstrates an end-to-end DevOps lifecycle for an AI workload:

```text
Source Code
    ↓
GitHub
    ↓
CI Validation
    ↓
Security Scanning
    ↓
Helm Packaging
    ↓
Kubernetes Deployment
    ↓
Persistent Storage
    ↓
Health Checks
    ↓
Monitoring
    ↓
Alerting
    ↓
Continuous Deployment
    ↓
Operational Troubleshooting
```

The focus is on **deploying and operating AI infrastructure**, rather than building an AI model.
