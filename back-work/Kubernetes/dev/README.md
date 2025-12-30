my-project/
├── README.md                          # Entry point: overview of platform, architecture, and workflows
│
├── apps/                              # Application source code (build-time concerns)
│   └── backend/                       # Backend application
│       ├── Dockerfile                 # Docker image definition for backend service
│       ├── src/                       # Application source code
│       └── README.md                  # Backend-specific documentation
│
├── helm/                              # Helm charts (deployment templates)
│   └── backend/                       # Helm chart for backend service
│       ├── Chart.yaml                 # Helm chart metadata (name, version)
│       ├── values.yaml                # Default values (shared across envs)
│       ├── values-dev.yaml            # Dev environment overrides
│       ├── values-prod.yaml           # Prod environment overrides
│       └── templates/                 # Kubernetes manifests rendered by Helm
│           ├── deployment.yaml        # Kubernetes Deployment for backend pods
│           ├── service.yaml           # ClusterIP/LoadBalancer service
│           ├── hpa.yaml               # Horizontal Pod Autoscaler
│           ├── ingress.yaml           # Ingress / HTTP(S) exposure
│           ├── serviceaccount.yaml    # K8s ServiceAccount (Workload Identity)
│           └── secret.yaml             # Secret references (Secret Manager / external)
│
├── argocd/                            # GitOps layer (continuous deployment)
│   ├── applications/                  # ArgoCD Application definitions
│   │   └── backend-app.yaml           # ArgoCD app pointing to Helm backend chart
│   └── projects/                     # ArgoCD AppProject definitions
│       └── platform-project.yaml      # Project-level RBAC, namespaces, repos
│
├── ci/                                # CI workflows (build-time only)
│   └── github-actions/               # GitHub Actions workflows
│       └── docker-build-push.yaml     # Build Docker image & push to Artifact Registry
│
├── monitoring/                        # Observability stack
│   ├── prometheus/                   # Prometheus monitoring configs
│   │   ├── servicemonitor.yaml       # Prometheus ServiceMonitor for backend metrics
│   │   └── alerts.yaml               # Alerting rules (CPU, memory, latency, errors)
│   └── grafana/                      # Grafana dashboards
│       └── dashboards/
│           └── backend-dashboard.json # Visual dashboard for backend metrics
│
├── environments/                      # Environment-specific configuration
│   ├── dev/                           # Development environment
│   │   └── values.yaml                # Dev-specific Helm values
│   └── prod/                          # Production environment
│       └── values.yaml                # Prod-specific Helm values
│
├── docs/                              # Platform documentation
│   ├── architecture.md                # GCP + GKE architecture explanation
│   ├── gke-setup.md                   # Step-by-step GKE cluster setup
│   └── gitops-flow.md                 # GitOps workflow (CI → ArgoCD → GKE)
