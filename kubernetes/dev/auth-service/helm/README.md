# Auth Service Helm Chart

This Helm chart deploys the auth-service application to Kubernetes.

## Installation

### Install the chart

```bash
helm install auth-service . --namespace auth-dev --create-namespace
```

### Upgrade the chart

```bash
helm upgrade auth-service . --namespace auth-dev
```

### Uninstall the chart

```bash
helm uninstall auth-service --namespace auth-dev
```

## Configuration

The following table lists the configurable parameters and their default values:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `namespace.name` | Namespace name | `auth-dev` |
| `namespace.create` | Create namespace | `true` |
| `image.repository` | Image repository | `us-east4-docker.pkg.dev/rapidqs-lightspeed/ci-cd-repo/auth-service` |
| `image.tag` | Image tag | `c7886c9320e03d0d5648fb25bc8187c3132b02db` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `replicaCount` | Number of replicas | `2` |
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.name` | Service account name | `gke-sa` |
| `serviceAccount.annotations` | Service account annotations | See values.yaml |
| `service.type` | Service type | `ClusterIP` |
| `sealedSecret.create` | Create sealed secret | `true` |
| `sealedSecret.name` | Sealed secret name | `auth-service-secrets` |

## Customization

You can override values by:

1. Creating a custom values file:
```bash
helm install auth-service . -f my-values.yaml
```

2. Using `--set` flag:
```bash
helm install auth-service . --set replicaCount=3
```

## Example: Updating Image Tag

```bash
helm upgrade auth-service . --set image.tag=NEW_TAG --namespace auth-dev
```

