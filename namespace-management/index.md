# Namespace Management

This section provides comprehensive guidance on managing Kubernetes namespaces, including creation, maintenance, troubleshooting, and cleanup procedures.

## Overview

Namespaces in Kubernetes provide a mechanism for isolating groups of resources within a single cluster. They are essential for multi-tenant environments, allowing teams to work in virtual clusters within the same physical cluster. Proper namespace management is crucial for maintaining a clean, efficient, and secure Kubernetes environment.

## Quick Navigation

- [Namespace Cleanup Procedures](./cleanup.md)
- [Namespace Troubleshooting](./troubleshooting.md)

## Namespace Basics

### Creating Namespaces

```bash
# Create a new namespace
kubectl create namespace <namespace-name>

# Create a namespace with labels
kubectl create namespace <namespace-name> --labels=environment=dev,team=frontend
```

### Resource Visibility and Scope

Namespaces only isolate namespaced resources. Cluster-wide resources are visible across all namespaces:

**Namespaced Resources:**
- Pods, Services, Deployments
- ConfigMaps, Secrets
- PersistentVolumeClaims
- ServiceAccounts, Roles, RoleBindings

**Cluster-wide Resources:**
- Nodes, PersistentVolumes
- ClusterRoles, ClusterRoleBindings
- Namespaces themselves
- CustomResourceDefinitions

## Namespace Management Best Practices

### Resource Organization

1. **Use Consistent Naming Conventions**: Adopt clear, consistent naming schemes
2. **Apply Labels and Annotations**: Add metadata to facilitate filtering and automation
3. **Set Resource Quotas**: Limit resource consumption per namespace
4. **Define Limit Ranges**: Set default resource limits for containers

### Security Considerations

1. **Implement RBAC**: Use Role-Based Access Control to restrict access
2. **Network Policies**: Control traffic between namespaces
3. **Service Accounts**: Create dedicated service accounts for each application
4. **Resource Isolation**: Prevent resources in one namespace from affecting others

### Example: Setting Resource Quotas

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
  namespace: <namespace-name>
spec:
  hard:
    pods: "20"
    requests.cpu: "2"
    requests.memory: 2Gi
    limits.cpu: "4"
    limits.memory: 4Gi
```

### Example: Setting Limit Ranges

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: <namespace-name>
spec:
  limits:
  - default:
      memory: 512Mi
      cpu: 500m
    defaultRequest:
      memory: 256Mi
      cpu: 200m
    type: Container
```

## Common Namespace Operations

### Switching Between Namespaces

```bash
# Set default namespace for current context
kubectl config set-context --current --namespace=<namespace-name>

# View resources in specific namespace
kubectl get pods -n <namespace-name>
```

### Monitoring Namespace Resources

```bash
# List all resources in a namespace
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get -n <namespace-name> --show-kind --ignore-not-found

# Check resource usage
kubectl top pod -n <namespace-name>
```

### Namespace Cleanup

For detailed cleanup procedures, see the [Namespace Cleanup guide](./cleanup.md).

Quick cleanup command:
```bash
kubectl delete namespace <namespace-name>
```

### Troubleshooting Namespace Issues

For comprehensive troubleshooting steps, see the [Namespace Troubleshooting guide](./troubleshooting.md).

Common troubleshooting command:
```bash
# Check namespace status
kubectl describe namespace <namespace-name>
```

## Advanced Namespace Management

### Using Labels for Organization

```bash
# Add labels to namespace
kubectl label namespace <namespace-name> environment=prod team=frontend

# List namespaces with specific label
kubectl get namespaces -l environment=prod
```

### Network Policies

Sample network policy to restrict access between namespaces:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-from-other-namespaces
  namespace: <namespace-name>
spec:
  podSelector: {}
  ingress:
  - from:
    - podSelector: {}
```

### Namespace Budget Planning

1. **Audit Current Usage**: Regularly review resource consumption
2. **Forecast Growth**: Plan for application scaling
3. **Implement Monitoring**: Set up alerts for quota thresholds
4. **Regular Reviews**: Periodically adjust quotas based on actual usage

## Tools for Namespace Management

1. **[kubectx & kubens](https://github.com/ahmetb/kubectx/tree/master)**: Quickly switch between namespaces and contexts
2. **[kyverno](https://kyverno.io)**: Policy management for namespace governance

## See Also

- [Resource Management](../resource-management/index.md)
- [Working with Finalizers](../resource-management/finalizers.md)
- [Reference Commands](../reference/commands.md)

![hypermedia tech logo](../assets/images/hypermediatech-default.webp)