# Deployment Flow (GitOps)

1. Developer pushes code to GitHub
2. GitHub Actions:
   - Builds Docker image
   - Pushes to Artifact Registry
3. Image tag updated in Helm values
4. ArgoCD detects Git change
5. ArgoCD syncs Helm chart
6. Kubernetes Deployment updated
7. HPA scales pods based on metrics

