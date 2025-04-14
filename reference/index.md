# Reference Guide

This section provides quick reference materials for Kubernetes cluster operations, including commonly used commands, helpful tools, and best practices for everyday tasks.

## Overview

The reference guides in this section are designed to help cluster operators and developers quickly find information they need for common tasks without needing to search through lengthy documentation.

## Quick Navigation

### Commands Reference

The [Commands Reference](./commands.md) provides:
- Frequently used kubectl commands organized by category
- Common command patterns with explanations
- One-liners for complex operations
- Command flags and options reference

Visit the [Commands Reference](./commands.md) for detailed command examples.

### Tools Reference

The [Tools Reference](./tools.md) provides:
- Essential tools for cluster management
- Installation and setup instructions
- Common use cases for each tool
- Integration with existing workflows

Visit the [Tools Reference](./tools.md) to learn about useful tools for Kubernetes operations.

## Common Quick Reference

### Kubernetes API Resources

```bash
# List all API resources
kubectl api-resources

# Get details about a specific resource type
kubectl explain <resource>

# Examples:
kubectl explain pod
kubectl explain deployment.spec.strategy
```

### Context Management

```bash
# List available contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# Set namespace for current context
kubectl config set-context --current --namespace=<namespace>
```

### Getting Help

```bash
# Get kubectl help
kubectl --help

# Get help for specific commands
kubectl <command> --help

# Kubectl cheat sheet
kubectl --help | grep cheat
```

## Best Practices Quick Reference

### Command Line Efficiency

1. **Use shortcuts**: `kubectl get po` instead of `kubectl get pods`
2. **Set default namespace**: `kubectl config set-context --current --namespace=<namespace>`
3. **Use kubectl aliases**: Create aliases for common commands
4. **Use kubectl plugins**: Install useful plugins with krew

### Output Formatting

```bash
# Get output in different formats
kubectl get pods -o wide
kubectl get pods -o yaml
kubectl get pods -o json
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase

# Sort output
kubectl get pods --sort-by=.metadata.creationTimestamp
```

## See Also

- [Namespace Management](../namespace-management/index.md)
- [Resource Management](../resource-management/index.md)
- [ArgoCD Operations](../argocd/index.md)