# Understanding and Managing Finalizers in Kubernetes

Finalizers are an important concept in Kubernetes resource management that can sometimes cause confusion or operational issues. This guide explains what finalizers are, how they work, and provides practical commands for managing them.

## What Are Finalizers?

Finalizers are metadata keys on Kubernetes resources that prevent resources from being deleted until specific cleanup operations have completed. They act as a protection mechanism to ensure orderly cleanup when resources are deleted.

When you attempt to delete a resource with finalizers:
1. Kubernetes marks the resource for deletion (sets a `deletionTimestamp`)
2. The resource remains in a "Terminating" state
3. The controllers responsible for each finalizer perform their cleanup operations
4. Once a controller completes its cleanup, it removes its finalizer
5. When all finalizers are removed, the resource is finally deleted

Common finalizers include:
- `kubernetes.io/pv-protection`
- `foregroundDeletion`
- `kubernetes.io/pvc-protection`
- `argocd.argoproj.io/finalizer` (for ArgoCD resources)

## When Finalizers Become Problematic

Finalizers can sometimes cause resources to become "stuck" in a terminating state. This typically happens when:

- The controller responsible for the finalizer is missing or malfunctioning
- The cleanup process encounters errors
- Custom controllers have been improperly implemented
- A CRD has been deleted before its instances

## Identifying Resources with Finalizers

### Check a Specific Resource

```bash
kubectl get <resource-type> <resource-name> -n <namespace> -o jsonpath='{.metadata.finalizers}'
```

Example:
```bash
kubectl get pod my-pod -n default -o jsonpath='{.metadata.finalizers}'
```

### List All Resources with Finalizers in a Namespace

```bash
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 -I{} bash -c "kubectl get {} -n <namespace> -o json | jq '.items[] | select(.metadata.finalizers != null and .metadata.finalizers | length > 0) | \"\(.kind) \(.metadata.name) has finalizers: \(.metadata.finalizers)\"'"
```

### Check If Resources Are Stuck in Terminating State

```bash
kubectl get all -n <namespace> | grep Terminating
```

## Removing Finalizers

⚠️ **CAUTION**: Forcibly removing finalizers bypasses normal cleanup processes. Only do this when you're certain the controllers responsible won't act on them or when a resource is genuinely stuck.

### Remove Finalizers from a Specific Resource

```bash
kubectl patch <resource-type> <resource-name> -n <namespace> --type='json' -p='[{"op":"remove","path":"/metadata/finalizers"}]'
```

Example:
```bash
kubectl patch pod stuck-pod -n default --type='json' -p='[{"op":"remove","path":"/metadata/finalizers"}]'
```

### Remove Finalizers from All Resources of a Specific Type

```bash
kubectl get <resource-type> -n <namespace> -o name | xargs -I{} kubectl patch {} -n <namespace> --type='json' -p='[{"op":"remove","path":"/metadata/finalizers"}]'
```

Example for ApplicationSets in ArgoCD:
```bash
kubectl -n argocd get applicationset -o name | xargs -I{} kubectl -n argocd patch {} --type='json' -p='[{"op":"remove","path":"/metadata/finalizers"}]'
```

### Alternative Method: Edit the Resource Directly

```bash
kubectl edit <resource-type> <resource-name> -n <namespace>
```

Then manually remove the finalizers array in the metadata section.

## Force Deleting a Namespace with Stuck Resources

If a namespace is stuck in Terminating state:

```bash
kubectl get namespace <namespace-name> -o json | jq '.spec.finalizers = []' > ns.json
kubectl replace --raw "/api/v1/namespaces/<namespace-name>/finalize" -f ns.json
```

## Best Practices

1. **Investigate First**: Before removing finalizers, understand why they're not being removed normally
2. **Document Actions**: Keep a record of manual interventions in your runbooks
3. **Check for Orphaned Resources**: Finalizers often protect against orphaned dependent resources
4. **Backup**: If possible, take resource manifests or cluster snapshots before force removing finalizers
5. **Review Controllers**: Recurring issues may indicate a problem with your controllers or configuration

## See Also
- [Namespace Cleanup Guide](../namespace-management/cleanup.md)
- [Common kubectl Commands](../reference/commands.md)


![hypermedia tech logo](../assets/images/hypermediatech-default.webp)