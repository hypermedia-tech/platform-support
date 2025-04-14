# Namespace Cleanup Procedures

This guide provides step-by-step instructions for cleaning up Kubernetes namespaces that may be stuck or containing resources that prevent proper deletion.

## Overview

Kubernetes namespaces occasionally become difficult to delete due to resources with finalizers, stuck controllers, or other issues. This guide provides systematic approaches to identify and resolve these issues.

## Standard Namespace Deletion

Under normal circumstances, deleting a namespace should be straightforward:

```bash
kubectl delete namespace <namespace-name>
```

However, if the namespace becomes stuck in the "Terminating" state, you'll need the procedures outlined below.

## Identifying Stuck Resources

### Check Namespace Status

First, verify the namespace is actually stuck:

```bash
kubectl get namespace <namespace-name>
```

If it shows `Terminating` status for more than a few minutes, it's likely stuck.

### List All Resources in the Namespace

To identify what resources might be preventing deletion:

```bash
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n <namespace-name>
```

### Identify Resources with Finalizers

Finalizers often prevent namespace deletion. Find resources with finalizers:

```bash
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 -I{} bash -c "kubectl get {} -n <namespace-name> -o json | jq '.items[] | select(.metadata.finalizers != null and .metadata.finalizers | length > 0) | \"\(.kind) \(.metadata.name) has finalizers: \(.metadata.finalizers)\"'"
```

## Cleanup Procedures

### Method 1: Remove Finalizers from Resources

For each resource with finalizers:

```bash
kubectl patch <resource-type> <resource-name> -n <namespace-name> --type='json' -p='[{"op":"remove","path":"/metadata/finalizers"}]'
```

Example:

```bash
kubectl patch deployment stuck-deployment -n stuck-namespace --type='json' -p='[{"op":"remove","path":"/metadata/finalizers"}]'
```

### Method 2: Force Remove Finalizers from Namespace

If the namespace itself has finalizers:

```bash
kubectl get namespace <namespace-name> -o json | jq '.spec.finalizers = []' > ns.json
kubectl replace --raw "/api/v1/namespaces/<namespace-name>/finalize" -f ns.json
```

### Method 3: Remove Specific Known Problematic Resources

#### Clear ArgoCD Applications

```bash
kubectl -n <namespace-name> get applications -o name | xargs -I{} kubectl patch {} -n <namespace-name> --type='json' -p='[{"op":"remove","path":"/metadata/finalizers"}]'
```

#### Clear Custom Resources

If you know specific CRDs causing problems:

```bash
kubectl get <crd-type> -n <namespace-name> -o name | xargs -I{} kubectl patch {} -n <namespace-name> --type='json' -p='[{"op":"remove","path":"/metadata/finalizers"}]'
```

#### Remove Problematic Pods

```bash
kubectl get pod -n <namespace-name> -o name | xargs -I{} kubectl delete {} -n <namespace-name> --force --grace-period=0
```

## Handling Special Cases

### Persistent Volumes and Claims

PVCs often prevent namespace deletion:

```bash
# List PVCs in the namespace
kubectl get pvc -n <namespace-name>

# Remove finalizers from PVCs
kubectl get pvc -n <namespace-name> -o name | xargs -I{} kubectl patch {} -n <namespace-name> --type='json' -p='[{"op":"remove","path":"/metadata/finalizers"}]'
```

### Webhook Configurations

If validating or mutating webhooks are causing issues:

```bash
# List webhook configurations
kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations

# Delete specific webhook if needed
kubectl delete validatingwebhookconfigurations <webhook-name>
```

### Service Account Tokens

```bash
kubectl get secrets -n <namespace-name> | grep service-account-token | awk '{print $1}' | xargs -I{} kubectl delete secret {} -n <namespace-name>
```

## Namespace Recovery Scripts

For batch cleanup operations, here's a more comprehensive script:

```bash
#!/bin/bash
NAMESPACE=$1

# Print namespace status
echo "Namespace status:"
kubectl get namespace $NAMESPACE -o yaml

# Get all resources with finalizers
echo "Resources with finalizers:"
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 -I{} bash -c "kubectl get {} -n $NAMESPACE -o json 2>/dev/null | jq -r '.items[] | select(.metadata.finalizers != null and .metadata.finalizers | length > 0) | \"\(.kind) \(.metadata.name) has finalizers: \(.metadata.finalizers)\"'"

# Remove finalizers from all resources
echo "Removing finalizers from resources..."
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 -I{} bash -c "kubectl get {} -n $NAMESPACE -o json 2>/dev/null | jq -r '.items[] | select(.metadata.finalizers != null and .metadata.finalizers | length > 0) | \"\(.kind)/\(.metadata.name)\"'" | xargs -I{} kubectl patch {} -n $NAMESPACE --type='json' -p='[{"op":"remove","path":"/metadata/finalizers"}]'

# Force delete pods
echo "Force deleting pods..."
kubectl get pods -n $NAMESPACE -o name | xargs -I{} kubectl delete {} -n $NAMESPACE --force --grace-period=0

# Remove namespace finalizers
echo "Removing namespace finalizers..."
kubectl get namespace $NAMESPACE -o json | jq '.spec.finalizers = []' > ns.json
kubectl replace --raw "/api/v1/namespaces/$NAMESPACE/finalize" -f ns.json

echo "Namespace status after cleanup:"
kubectl get namespace $NAMESPACE -o yaml
```

Save this as `cleanup-namespace.sh` and run with `./cleanup-namespace.sh <namespace-name>`.

## Prevention Best Practices

To avoid stuck namespaces in the future:

1. **Delete Resources Before Namespace**: Delete key resources before deleting the namespace
2. **Use Resource Quotas**: Set resource quotas to limit the number of resources
3. **Implement Namespace Lifecycle Policies**: Create policies for namespace creation and deletion
4. **Regular Maintenance**: Periodically check for orphaned resources
5. **Use Labels**: Label resources consistently for easier cleanup

## See Also

- [Namespace Troubleshooting](./troubleshooting.md)
- [Working with Finalizers](../resource-management/finalizers.md)
- [ArgoCD Application Management](../argocd/application-management.md)