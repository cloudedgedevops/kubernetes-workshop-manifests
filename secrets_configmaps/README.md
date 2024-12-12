# Kubernetes Secrets and ConfigMap Workshop

## Objective
Demonstrate how to use Kubernetes ConfigMaps and Secrets, including an Azure Key Vault integration example.

## Prerequisites
- Kubernetes cluster with Azure Key Vault CSI driver
- kubectl installed
- Basic understanding of Kubernetes concepts
- Azure Key Vault setup (for Azure Key Vault example)

## Repository Structure
```
.
├── configmap.yaml
├── secret.yaml
├── azure_kv.yaml
└── pod.yaml
```

## Workshop Walkthrough

### 1. Create Namespace
```bash
# Create the namespace for our resources
kubectl create namespace secret-configmap-namespace
```

### 2. ConfigMap Demonstration

#### Create ConfigMap
```bash
# Apply the ConfigMap
kubectl apply -f configmap.yaml

# Verify ConfigMap creation
kubectl get configmap -n secret-configmap-namespace
kubectl describe configmap demo-configmap -n secret-configmap-namespace
```

### 3. Secrets Demonstration

#### Create Secret
```bash
# Apply the Secret
kubectl apply -f secret.yaml

# Verify Secret creation
kubectl get secrets -n secret-configmap-namespace
# Note: Secrets are intentionally obfuscated
```

### 4. Pod Configuration with ConfigMap and Secret

#### Deploy Pod
```bash
# Apply the Pod configuration
kubectl apply -f pod.yaml

# Check Pod status
kubectl get pods -n secret-configmap-namespace
```

### 5. Exploring Secrets and ConfigMaps

#### Verify Environment Variables
```bash
# Exec into the pod to check environment variables
kubectl exec -it demo-pod -n secret-configmap-namespace -- env | grep CONFIG_KEY
kubectl exec -it demo-pod -n secret-configmap-namespace -- env | grep SECRET_PASSWORD
```

### 6. Azure Key Vault Integration

#### Deploy SecretProviderClass
```bash
# Apply the SecretProviderClass
kubectl apply -f azure_kv.yaml

# Verify SecretProviderClass
kubectl get secretproviderclass -n secret-configmap-namespace
```

## Workshop Challenges

1. **Create a Custom ConfigMap**
   - Create a ConfigMap with multiple key-value pairs
   - Use it in a pod as environment variables and volume mount

2. **Secret Management**
   - Create a secret from a literal value
   - Create a secret from a file
   - Discuss best practices for secret management

3. **Azure Key Vault Integration**
   - Configure SecretProviderClass with your own Key Vault
   - Demonstrate retrieving secrets from Azure Key Vault

## Best Practices

### ConfigMap Best Practices
- Use ConfigMaps for non-sensitive configuration
- Separate configuration from container images
- Can be updated without rebuilding containers

### Secret Best Practices
- Never commit secrets to version control
- Use external secret management tools
- Rotate secrets regularly
- Use RBAC to limit secret access

### Azure Key Vault Integration
- Use managed identities
- Implement least privilege access
- Rotate secrets and certificates regularly

## Advanced Topics
- Sealed Secrets
- External Secret Operators
- Vault integration
- Kubernetes Secret Encryption

## Clean Up
```bash
# Remove all resources
kubectl delete -f .
# Or delete the namespace
kubectl delete namespace secret-configmap-namespace
```