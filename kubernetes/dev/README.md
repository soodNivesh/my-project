# Dev Environment Kubernetes Manifests

This directory contains Kubernetes manifests and configurations for the **development environment** of the Lightspeed application. **Services are deployed using Helm charts** via ArgoCD (GitOps). The manifests are organized by service and also include Kustomize configurations for alternative deployment methods.

## Directory Structure

```
kubernetes/dev/
├── README.md                          # This file
├── argocd/                            # ArgoCD GitOps configurations
│   ├── project.yaml                   # ArgoCD AppProject for dev environment
│   ├── apps.yaml                      # Main ArgoCD Application definition
│   └── kustomize/                     # Kustomize-based ArgoCD applications
│       ├── auth-service-app.yaml      # Auth service ArgoCD app
│       └── project-service-app.yaml   # Project service ArgoCD app
├── auth-service/                      # Authentication service
│   ├── deployment.yaml                # Kubernetes Deployment manifest
│   ├── service-account.yaml           # Service account for Workload Identity
│   ├── sealed-secrets.yaml            # SealedSecret for sensitive data
│   ├── kustomization.yaml             # Kustomize configuration
│   └── helm/                          # Helm chart for auth-service
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── namespace.yaml
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── serviceaccount.yaml
│           └── sealed-secret.yaml
├── project-service/                   # Project management service
│   ├── deployment.yaml                # Kubernetes Deployment manifest
│   ├── service-account.yaml           # Service account for Workload Identity
│   ├── sealed-secrets.yaml            # SealedSecret for sensitive data
│   ├── kustomization.yaml             # Kustomize configuration
│   └── helm/                          # Helm chart for project-service
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── namespace.yaml
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── serviceaccount.yaml
│           └── sealed-secret.yaml
└── external-gateway-service/          # External API Gateway (Gateway API)
    ├── external-gateway.yaml          # Gateway resource
    ├── auth-route.yaml                # Route for auth service
    ├── project-route.yaml             # Route for project service
    ├── redirect-route.yaml            # Redirect rules
    ├── certificate.yaml               # Certificate Manager certificate
    ├── dns-authorization.yaml         # DNS authorization for cert
    ├── auth-referencegrant.yaml       # ReferenceGrant for auth service
    └── project-referencegrant.yaml    # ReferenceGrant for project service
```

## Architecture Overview

The dev environment consists of:

### Services

1. **Auth Service** (`auth-service`)
   - Namespace: `auth-dev`
   - Ports: HTTP (3000), gRPC (50051)
   - Health endpoint: `/auth/health`
   - Replicas: 2
   - Image registry: `us-east4-docker.pkg.dev/rapidqs-lightspeed/ci-cd-repo/auth-service`

2. **Project Service** (`project-service`)
   - Namespace: `project-dev`
   - Ports: HTTP (3001)
   - Health endpoint: `/project/health`
   - Replicas: 2
   - Image registry: `us-east4-docker.pkg.dev/rapidqs-lightspeed/ci-cd-repo/project-service`

3. **External Gateway Service** (`external-gateway-service`)
   - Namespace: `ingress-dev`
   - Gateway class: `gke-l7-regional-external-managed`
   - Hostname: `api.lightspeed.rapidqs.com`
   - Protocols: HTTP (80), HTTPS (443)
   - Uses Google Cloud Certificate Manager for TLS

### Key Features

- **Helm-based Deployment**: Services are deployed using Helm charts
- **GitOps with ArgoCD**: Automated deployments from Git repository using Helm charts
- **Sealed Secrets**: Encrypted secrets using Bitnami Sealed Secrets
- **Workload Identity**: GKE Service Accounts for secure GCP access
- **Gateway API**: Modern ingress using Kubernetes Gateway API
- **Health Checks**: Readiness and liveness probes configured
- **Multi-format Support**: Helm charts (primary) and Kustomize configurations (alternative) available

## Deployment Methods

### Method 1: ArgoCD with Helm (Recommended - GitOps)

**Services are deployed using Helm charts** via ArgoCD. ArgoCD automatically syncs changes from the Git repository to the cluster using the Helm charts located in each service's `helm/` directory.

#### Prerequisites

1. ArgoCD installed in the cluster
2. Repository access configured in ArgoCD
3. ArgoCD project created (see `argocd/project.yaml`)

#### Accessing ArgoCD UI

##### Get ArgoCD UI URL

Get the ArgoCD UI URL through the service:

```bash
# Check if ArgoCD server has an external IP or LoadBalancer
kubectl get svc argocd-server -n argocd

# If using LoadBalancer, get the external IP
kubectl get svc argocd-server -n argocd -o jsonpath='{.status.loadBalancer.ingress[0].ip}'

# If using Ingress, get the ingress hostname
kubectl get ingress argocd-server -n argocd
```

##### Get ArgoCD Username and Password

**Default Username:**
- The default ArgoCD username is: `admin`

**Get Initial Password:**

The initial password is stored in a Kubernetes secret. Retrieve it using:

```bash
# Get the initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

**Change Password (Optional):**

After first login, you can change the password:

```bash
# Login via CLI
argocd login <argocd-server-address>

# Change password
argocd account update-password
```

Or change it via the UI:
1. Login to ArgoCD UI
2. Click on your username (top right) → **Account Settings**
3. Click **Update Password**
4. Enter current password and new password

**Create Additional Users (Optional):**

```bash
# Create a new user account
argocd account create-password <username> --password <password>

# Or via kubectl (create secret)
kubectl create secret generic argocd-<username>-secret \
  --from-literal=password=<bcrypt-hashed-password> \
  -n argocd
```

##### Login to ArgoCD UI

1. Open the ArgoCD UI URL in your browser
2. Login with:
   - **Username**: `admin`
   - **Password**: (retrieved from the secret above)

#### Deploy via ArgoCD

The ArgoCD applications are configured to deploy services using **Helm charts**. Each service application points to the Helm chart directory:

- `auth-service-app.yaml` → `kubernetes/dev/auth-service/helm`
- `project-service-app.yaml` → `kubernetes/dev/project-service/helm`

```bash
# Apply the ArgoCD project
kubectl apply -f argocd/project.yaml

# Apply the main ArgoCD application
kubectl apply -f argocd/apps.yaml

# Or apply individual service applications (these use Helm charts)
kubectl apply -f argocd/kustomize/auth-service-app.yaml
kubectl apply -f argocd/kustomize/project-service-app.yaml
```

#### Verify ArgoCD Sync

```bash
# Check ArgoCD application status
kubectl get applications -n argocd

# View detailed status
argocd app get dev -n argocd

# View sync status
argocd app sync dev -n argocd
```

### Method 2: Helm (Direct)

Deploy directly using Helm charts (without ArgoCD):

```bash
# Deploy auth service
helm install auth-service auth-service/helm -n auth-dev --create-namespace

# Deploy project service
helm install project-service project-service/helm -n project-dev --create-namespace
```

### Method 3: Kustomize (Alternative)

Deploy directly using Kustomize (alternative method, not used by ArgoCD):

```bash
# Deploy auth service
kubectl apply -k auth-service/

# Deploy project service
kubectl apply -k project-service/

# Deploy external gateway
kubectl apply -f external-gateway-service/
```

## Configuration

### Updating Container Images

#### Using Helm (Primary Method)

Since services are deployed using Helm charts, update the image in `helm/values.yaml`:

```yaml
image:
  repository: us-east4-docker.pkg.dev/rapidqs-lightspeed/ci-cd-repo/auth-service
  tag: NEW_TAG
```

Then upgrade:
```bash
helm upgrade auth-service auth-service/helm -n auth-dev
```

If using ArgoCD, commit the changes to Git and ArgoCD will automatically sync the updated Helm chart.

#### Using Kustomize (Alternative Method)

Edit the `kustomization.yaml` file in the service directory:

```yaml
images:
- name: auth-service
  newName: us-east4-docker.pkg.dev/rapidqs-lightspeed/ci-cd-repo/auth-service:NEW_TAG
```

Then apply:
```bash
kubectl apply -k auth-service/
```

**Note**: This method is not used by ArgoCD. ArgoCD uses Helm charts for deployment.

### Managing Secrets

Secrets are managed using **Sealed Secrets**. To update secrets:

1. Install `kubeseal` CLI tool
2. Create a sealed secret from your secret:

```bash
# Create a Kubernetes secret
kubectl create secret generic auth-service-secrets \
  --from-literal=DB_PASSWORD=yourpassword \
  --dry-run=client -o yaml | kubeseal -o yaml > auth-service/sealed-secrets.yaml
```

3. Apply the sealed secret:
```bash
kubectl apply -f auth-service/sealed-secrets.yaml
```

The Sealed Secrets controller will automatically create the Kubernetes secret.

### Gateway Configuration

The external gateway is configured to:
- Use Google Cloud Certificate Manager for TLS certificates
- Route traffic to `api.lightspeed.rapidqs.com`
- Support both HTTP and HTTPS protocols
- Use GKE managed Gateway class

To update the hostname, edit `external-gateway-service/external-gateway.yaml`:

```yaml
spec:
  listeners:
    - name: https
      hostname: your-new-hostname.com
```

## Common Operations

### Check Service Status

```bash
# Check all pods
kubectl get pods -n auth-dev
kubectl get pods -n project-dev

# Check services
kubectl get svc -n auth-dev
kubectl get svc -n project-dev

# Check deployments
kubectl get deployments -n auth-dev
kubectl get deployments -n project-dev
```

### View Logs

```bash
# Auth service logs
kubectl logs -n auth-dev -l app=auth-service --tail=100

# Project service logs
kubectl logs -n project-dev -l app=project-service --tail=100

# Follow logs
kubectl logs -n auth-dev -l app=auth-service -f
```

### Scale Services

```bash
# Scale auth service
kubectl scale deployment auth-service -n auth-dev --replicas=3

# Scale project service
kubectl scale deployment project-service -n project-dev --replicas=3
```

### Update Deployments

```bash
# Restart deployment (rolling update)
kubectl rollout restart deployment/auth-service -n auth-dev
kubectl rollout restart deployment/project-service -n project-dev

# Check rollout status
kubectl rollout status deployment/auth-service -n auth-dev

# Rollback if needed
kubectl rollout undo deployment/auth-service -n auth-dev
```

### Gateway Operations

```bash
# Check gateway status
kubectl get gateway -n ingress-dev

# Check routes
kubectl get httproute -n ingress-dev

# Check certificate
kubectl get certificatemanagercertificate -n ingress-dev

# Get gateway IP address
kubectl get gateway external-api-gateway -n ingress-dev -o jsonpath='{.status.addresses[0].value}'
```

## Troubleshooting

### Pods Not Starting

```bash
# Check pod events
kubectl describe pod <pod-name> -n auth-dev

# Check pod logs
kubectl logs <pod-name> -n auth-dev

# Check if secrets exist
kubectl get secrets -n auth-dev
```

### Service Not Accessible

```bash
# Check service endpoints
kubectl get endpoints -n auth-dev

# Test service connectivity from within cluster
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
# Then: wget -O- http://auth-service.auth-dev.svc.cluster.local/auth/health
```

### Gateway Issues

```bash
# Check gateway status
kubectl describe gateway external-api-gateway -n ingress-dev

# Check certificate status
kubectl describe certificatemanagercertificate lightspeed-api-cert -n ingress-dev

# Check DNS authorization
kubectl get dnsauthorization -n ingress-dev
```

### ArgoCD Sync Issues

```bash
# Check ArgoCD application status
kubectl get application dev -n argocd -o yaml

# View ArgoCD logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller

# Force sync
argocd app sync dev -n argocd
```

## Security Considerations

1. **Sealed Secrets**: All sensitive data is encrypted using Sealed Secrets
2. **Workload Identity**: Services use GKE Service Accounts for secure GCP access
3. **Network Policies**: Consider enabling network policies for additional security
4. **TLS**: Gateway uses Certificate Manager for automatic TLS certificate management
5. **Private Services**: Services use ClusterIP (internal only) by default

## Additional Resources

- [Kubernetes Gateway API Documentation](https://gateway-api.sigs.k8s.io/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kustomize Documentation](https://kustomize.io/)
- [Helm Documentation](https://helm.sh/docs/)
- [Sealed Secrets Documentation](https://github.com/bitnami-labs/sealed-secrets)
- [GKE Gateway Controller](https://cloud.google.com/kubernetes-engine/docs/how-to/deploying-gateways)

## Migration Notes

**Services are deployed using Helm charts** as the primary method. The ArgoCD applications are configured to use Helm charts located in each service's `helm/` directory. Kustomize configurations are available as an alternative deployment method but are not used by ArgoCD.

To switch ArgoCD from Helm to Kustomize (not recommended):

1. Update `argocd/kustomize/*-app.yaml` files to point to the kustomization path
2. Change the `source.path` from `kubernetes/dev/{service}/helm` to `kubernetes/dev/{service}`
3. Apply the updated ArgoCD applications

