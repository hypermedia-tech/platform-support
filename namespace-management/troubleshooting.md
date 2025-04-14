# Namespace Troubleshooting

This guide provides solutions for common namespace-related issues in Kubernetes clusters.

## Common Namespace Issues

### Issue: Namespace Stuck in Terminating State

**Symptoms:**
- Namespace shows "Terminating" status for an extended period
- `kubectl delete namespace` command hangs or fails

**Diagnostic Commands:**

```bash
# Check namespace status
kubectl get namespace <namespace-name> -o yaml

# Check for finalizers on the namespace
kubectl get namespace <namespace-name> -o jsonpath='{.spec.finalizers}'
```

**Solutions:**

See the [Namespace Cleanup](./cleanup.md) guide for detailed procedures.

Quick solution:
```bash
kubectl get namespace <namespace-name> -o json | jq '.spec.finalizers = []' > ns.json
kubectl replace --raw "/api/v1/namespaces/<namespace-name>/finalize" -f ns.json
```

### Issue: Cannot Create Resources in Namespace

**Symptoms:**
- Error messages like "Error from server (Forbidden): error when creating..."
- Resources fail to create despite permissions

**Diagnostic Commands:**

```bash
# Check for resource quotas
kubectl get resourcequota -n <namespace-name>

# Check for limit ranges
kubectl get limitrange -n <namespace-name>

# Check for admission webhooks
kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations
```

**Solutions:**

- Adjust resource quotas:
  ```bash
  kubectl edit resourcequota <quota-name> -n <namespace-name>
  ```

- Check webhook configurations for issues:
  ```bash
  kubectl get validatingwebhookconfigurations -o yaml | grep <namespace-name>
  ```

### Issue: Namespace Not Showing Resources

**Symptoms:**
- Resources exist but don't appear in listing commands
- Inconsistent behavior between different kubectl commands

**Diagnostic Commands:**

```bash
# List all resource types in the namespace
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get -n <namespace-name> --show-kind --ignore-not-found
```

**Solutions:**

- Check RBAC permissions:
  ```bash
  kubectl auth can-i --list -n <namespace-name>
  ```

- Verify your current context:
  ```bash
  kubectl config current-context
  kubectl config view --minify
  ```

## Advanced Troubleshooting

### Network Policy Issues

If pods can't communicate within a namespace:

```bash
# List network policies
kubectl get networkpolicy -n <namespace-name>

# Test connectivity between pods
kubectl run test-$RANDOM --rm -it --image=alpine -n <namespace-name> -- sh -c "ping <pod-ip>"
```

### Event Monitoring

Monitor namespace events for clues:

```bash
# Watch namespace events
kubectl get events -n <namespace-name> --sort-by='.lastTimestamp'

# Monitor specific resource events
kubectl get events -n <namespace-name> --field-selector involvedObject.name=<resource-name>
```

### Resource Contention

If namespace has performance issues:

```bash
# Check resource usage
kubectl top pod -n <namespace-name>

# Check for pending pods
kubectl get pods -n <namespace-name> | grep Pending
```

## Checking Namespace Security

### Review RBAC Configuration

```bash
# List roles and bindings
kubectl get roles,rolebindings -n <namespace-name>

# Check who can do what
kubectl auth can-i --list --all-namespaces
```

### Network Policy Validation

```bash
# Check if network policies are enabled
kubectl get pods -n kube-system | grep -i network

# List all network policies affecting the namespace
kubectl get networkpolicy -A -o wide | grep <namespace-name>
```

## Namespace Auditing

### Resource Inventory

Take an inventory of namespace resources:

```bash
# Export all resources in a namespace
mkdir -p backup/<namespace-name>
kubectl api-resources --verbs=list --namespaced -o name | xargs -n1 -I{} sh -c "kubectl get {} -n <namespace-name> -o yaml > backup/<namespace-name>/{}.yaml"
```

### Resource Ownership Analysis

Find resources without proper ownership:

```bash
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 -I{} bash -c "kubectl get {} -n <namespace-name> -o json | jq -r '.items[] | select(.metadata.ownerReferences == null) | \"\(.kind) \(.metadata.name) has no owner\"'"
```

## Recovering from Serious Issues

### Creating a New Namespace with Same Resources

When a namespace is irreparably damaged:

1. Export all resources:
   ```bash
   kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 -I{} kubectl get {} -n <damaged-namespace> -o yaml > resources.yaml
   ```

2. Edit the resources.yaml file to clean up:
    - Remove `resourceVersion`, `uid`, and other auto-generated fields
    - Change namespace references to the new namespace
    - Remove finalizers

3. Create new namespace and apply resources:
   ```bash
   kubectl create namespace <new-namespace>
   kubectl apply -f cleaned-resources.yaml
   ```

### Recovering from Unauthorized Changes

If a namespace has been modified without authorization:

```bash
# Check recent changes
kubectl get events -n <namespace-name> --sort-by='.lastTimestamp'

# Review audit logs (if enabled in your cluster)
kubectl logs -n kube-system -l k8s-app=kube-apiserver --tail=1000 | grep <namespace-name>
```

## Best Practices

1. **Use Resource Limits**: Always set resource requests and limits
2. **Implement Network Policies**: Restrict traffic between namespaces
3. **Regular Auditing**: Periodically review namespace resources
4. **Use Labels and Annotations**: Consistently label resources for easier management
5. **Namespace RBAC**: Apply principle of least privilege for namespace access

## See Also

- [Namespace Cleanup Procedures](./cleanup.md)
- [Resource Management](../resource-management/index.md)
- [Working with Finalizers](../resource-management/finalizers.md)