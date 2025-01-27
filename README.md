# FHEVM Zama Helm Chart

This repository contains the Helm chart for deploying the Zama FHEVM development environment on Kubernetes. 

This chart is designed for use with ArgoCD, and all changes pushed to this repository are automatically merged into the ArgoCD configuration for deployment. To access the ArgoCD dashboard, use the following command:

```bash
kubectl port-forward deployment/argo-cd-argocd-server 8080:8080 -n argo-system
```

## Components

The chart deploys the following components:

- **FHEVM Validator**: Ethereum validator node with FHE capabilities
- **Gateway**: Blockchain gateway service
- **Gateway Store**: Storage service for the gateway
- **KMS Core**: Key Management Service core component
- **KMS Validator**: Blockchain validator for KMS
- **Connector**: Service connecting KMS components
- **Async Test CronJob**: Periodic async testing job
- **Test Prep CronJob**: Job for preparing test environments
- **EVM Processor**: Deployment for processing EVM-compatible data

## Prerequisites

- Kubernetes 1.16+
- Helm 3.0+
- PV provisioner support in the underlying infrastructure
- ArgoCD (for GitOps deployment)

## Installation

### Using ArgoCD

1. Create a namespace for the FHEVM components:
```bash
kubectl create namespace fhevm
```

2. Deploy using ArgoCD application manifest:
```bash
export GITHUB_REPO_URL="https://github.com/yourusername/fhevm-zama"
envsubst < argocd/applications/fhevm_zama.yaml | kubectl apply -f -
```

### Manual Installation

1. Clone the repository:
```bash
git clone https://github.com/yourusername/fhevm-zama.git
cd fhevm-zama
```

2. Install the chart:
```bash
helm install zama-dev chart/ -n fhevm
```

## Configuration

The following table lists the configurable parameters of the chart and their default values.

### Global Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.imageRegistry` | Global Docker image registry | `ghcr.io/zama-ai` |
| `global.imageTag` | Global Docker image tag | `v0.8.2-rc4` |

### FHEVM Validator Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `fhevmValidator.replicas` | Number of replicas | `1` |
| `fhevmValidator.env.TFHE_EXECUTOR_CONTRACT_ADDRESS` | Contract address | `0x05fD9B5EFE0a996095f42Ed7e77c390810CF660c` |

### Storage Configuration

The chart uses PersistentVolumeClaims for storing:
- Network FHE keys
- Gateway configuration
- KMS core keys
- Test preparation keys
- EVM block processing data

## Directory Structure

```plaintext
chart/templates/
├── _helpers.tpl
├── configmaps/
│   ├── fhevm-setup.yaml
│   ├── gateway-config.yaml
│   ├── async-test-configmap.yaml
│   └── test-prep-configmap.yaml
├── deployments/
│   ├── kms-validator-deployment.yaml
│   ├── connector-deployment.yaml
│   ├── gateway-store-deployment.yaml
│   ├── gateway-deployment.yaml
│   ├── fhevm-validator-deployment.yaml
│   ├── kms-core-deployment.yaml
│   └── evm-processor-deployment.yaml
├── jobs/
│   ├── async-test-cronjob.yaml
│   └── test-prep-cronjob.yaml
├── pvcs/
│   ├── kms-core-pvc.yaml
│   ├── gateway-pvc.yaml
│   ├── fhevm-validator-pvcs.yaml
│   ├── test-prep-keys-pvc.yaml
│   └── block-processor-pvc.yaml
├── rbac/
│   └── test-prep-cronjob.yaml
├── secrets/
│   └── test-prep-secrets.yaml
├── services/
│   ├── gateway-service.yaml
│   ├── gateway-store-service.yaml
│   ├── kms-core-service.yaml
│   ├── kms-validator-service.yaml
│   ├── fhevm-validator-service.yaml
│   └── evm-processor-service.yaml
```

## Security Considerations

- The chart includes init containers for secure key generation and distribution
- Configuration files are mounted using ConfigMaps
- All sensitive operations are performed within the cluster
- Container security contexts are properly configured

## Monitoring

The services expose metrics for monitoring:
- FHEVM Validator health checks
- KMS Core service health
- Gateway service status

## Troubleshooting

Common issues and solutions:

1. **Key Generation Failure**
   - Check init container logs:
   ```bash
   kubectl logs -n fhevm <pod-name> -c init-keys
   ```

2. **Service Connection Issues**
   - Verify service DNS resolution:
   ```bash
   kubectl exec -n fhevm <pod-name> -- nslookup kms-validator
   ```

3. **Volume Mount Problems**
   - Check PVC status:
   ```bash
   kubectl get pvc -n fhevm
   ```

## Development

To modify or extend this chart:

1. Update the templates in `chart/templates/`
2. Modify configuration in `values.yaml`
3. Test the changes:
```bash
helm template chart/ --debug
```


this config was built replicating fhevm-zama devops github: https://github.com/zama-ai/fhevm-devops/tree/release/0.5.x

There is no association with the zama team and this is just a built of a helm chart without any changes or additional interpretations of the code.

Please follow their licence from https://github.com/zama-ai/fhevm-devops/blob/release/0.5.x/LICENSE
