# Project Service Helm Chart

This Helm chart deploys the project-service application to Kubernetes.

## Installation

### Install the chart

```bash
helm install project-service . --namespace project-dev --create-namespace
```

### Upgrade the chart

```bash
helm upgrade project-service . --namespace project-dev
```

### Uninstall the chart

```bash
helm uninstall project-service --namespace project-dev
```

## Configuration

The following table lists the configurable parameters and their default values:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `namespace.name` | Namespace name | `project-dev` |
| `namespace.create` | Create namespace | `true` |
| `image.repository` | Image repository | `us-east4-docker.pkg.dev/rapidqs-lightspeed/ci-cd-repo/project-service` |
| `image.tag` | Image tag | `7cf851b00abdee39246ca40521e1c5ad9ce59df1` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `replicaCount` | Number of replicas | `2` |
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.name` | Service account name | `gke-sa` |
| `serviceAccount.annotations` | Service account annotations | See values.yaml |
| `service.type` | Service type | `ClusterIP` |
| `sealedSecret.create` | Create sealed secret | `true` |
| `sealedSecret.name` | Sealed secret name | `project-service-secrets` |

## Customization

You can override values by:

1. Creating a custom values file:
```bash
helm install project-service . -f my-values.yaml
```

2. Using `--set` flag:
```bash
helm install project-service . --set replicaCount=3
```

## Example: Updating Image Tag

```bash
helm upgrade project-service . --set image.tag=NEW_TAG --namespace project-dev
```

