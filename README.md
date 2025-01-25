# FHEVM Zama Helm Chart

This repository contains the Helm chart for deploying the Zama FHEVM development environment on Kubernetes.

## Components

The chart deploys the following components:

- **FHEVM Validator**: Ethereum validator node with FHE capabilities
- **Gateway**: Blockchain gateway service
- **Gateway Store**: Storage service for the gateway
- **KMS Core**: Key Management Service core component
- **KMS Validator**: Blockchain validator for KMS
- **Connector**: Service connecting KMS components

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

