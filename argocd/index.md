# ArgoCD Operations

This guide provides information on managing, troubleshooting, and operating ArgoCD in a Kubernetes environment.

## Overview

ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes that helps teams manage applications across multiple clusters. As a critical component in many Kubernetes environments, understanding how to effectively operate ArgoCD is essential for maintaining reliable deployments.

## Key Topics

### Application Management

The [Application Management](./application-management.md) guide covers:

- Creating and configuring Applications and ApplicationSets
- Managing application sync policies
- Bulk operations for efficient management
- Best practices for organizing applications

Learn how to efficiently manage your applications in our [detailed guide](./application-management.md).

### Troubleshooting

The [Troubleshooting](./troubleshooting.md) guide covers:

- Diagnosing and fixing sync failures
- Resolving resource health issues
- Recovering from serious failures
- Performance optimization techniques

When things go wrong, refer to our [troubleshooting guide](./troubleshooting.md) for solutions.

## Common Commands

Here are some frequently used commands for ArgoCD operations:

```bash
# Get all applications
kubectl get applications -n argocd

# Check application sync status
kubectl get applications -n argocd -o custom-columns=NAME:.metadata.name,SYNC:.status.sync.status,HEALTH:.status.health.status

# Restart ArgoCD components
kubectl rollout restart deployment -l app.kubernetes.io/part-of=argocd -n argocd

# Force refresh an application
kubectl patch application <app-name> -n argocd --type=merge -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}'
```

## Maintenance Tasks

### Regular Health Checks

Perform these checks regularly to ensure your ArgoCD installation is healthy:

1. Verify all ArgoCD pods are running:
   ```bash
   kubectl get pods -n argocd
   ```

2. Check for applications with sync issues:
   ```bash
   kubectl get applications -n argocd -o custom-columns=NAME:.metadata.name,SYNC:.status.sync.status | grep -v Synced
   ```

3. Monitor resource usage:
   ```bash
   kubectl top pod -n argocd
   ```

### Backup and Restore

It's recommended to regularly back up your ArgoCD configuration:

```bash
# Backup all ArgoCD resources
kubectl get all,applications,applicationsets,appprojects -n argocd -o yaml > argocd-backup.yaml
```

## Best Practices

1. **Use ApplicationSets**: Convert individual applications to ApplicationSets for better management
2. **Implement Projects**: Organize applications into logical groups with proper access controls
3. **Configure Resource Health Checks**: Customize health checks for your specific resource types
4. **Regular Auditing**: Review sync statuses and resource health regularly
5. **Performance Tuning**: Adjust resource limits and parallelism for large-scale deployments

## See Also

- [Namespace Management](../namespace-management/index.md)
- [Working with Finalizers](../resource-management/finalizers.md)
- [Common Kubectl Commands](../reference/commands.md)