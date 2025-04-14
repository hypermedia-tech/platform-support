# Common Kubernetes Resource Management Issues

This guide covers frequently encountered issues when managing Kubernetes resources and provides practical solutions to resolve them.

## Pod Issues

### Pods Stuck in Pending State

**Symptoms:**
- Pods remain in `Pending` status indefinitely
- Events show resource constraints or scheduling issues

**Diagnostic Commands:**
```bash
# Check pod status and events
kubectl describe pod <pod-name> -n <namespace>

# Check node resource availability
kubectl describe nodes | grep -A 5 "Allocated resources"
```

**Common Causes and Solutions:**

1. **Insufficient Resources**
    - Nodes don't have enough CPU/memory to schedule the pod
   ```bash
   # Check resource requests vs node capacity
   kubectl describe nodes | grep -A 10 "Capacity"
   
   # Adjust pod resource requests
   kubectl patch deployment <deployment-name> -n <namespace> --type=json -p='[{"op":"replace","path":"/spec/template/spec/containers/0/resources/requests","value":{"cpu":"100m","memory":"256Mi"}}]'
   ```

2. **Node Selector/Affinity Constraints**
    - Pod has node selectors that can't be satisfied
   ```bash
   # Check node labels
   kubectl get nodes --show-labels
   
   # Modify node selector if needed
   kubectl patch deployment <deployment-name> -n <namespace> --type=json -p='[{"op":"remove","path":"/spec/template/spec/nodeSelector"}]'
   ```

3. **PVC Binding Issues**
    - Pod requires PVC that can't be bound
   ```bash
   # Check PVC status
   kubectl get pvc -n <namespace>
   kubectl describe pvc <pvc-name> -n <namespace>
   ```

### Pods Stuck in Terminating State

**Symptoms:**
- Pod shows `Terminating` status for an extended period
- `kubectl delete pod` hangs

**Solutions:**

```bash
# Force delete the pod
kubectl delete pod <pod-name> -n <namespace> --force --grace-period=0

# If pod has finalizers, remove them
kubectl patch pod <pod-name> -n <namespace> --type='json' -p='[{"op":"remove","path":"/metadata/finalizers"}]'
```

### CrashLoopBackOff Errors

**Symptoms:**
- Pod status shows `CrashLoopBackOff`
- Container repeatedly restarts

**Diagnostic Commands:**
```bash
# Check pod logs
kubectl logs <pod-name> -n <namespace> --previous

# Check pod events
kubectl describe pod <pod-name> -n <namespace>
```

**Solutions:**

1. Fix application errors shown in logs
2. Ensure resource limits are adequate:
   ```bash
   kubectl patch deployment <deployment-name> -n <namespace> --type=json -p='[{"op":"replace","path":"/spec/template/spec/containers/0/resources/limits","value":{"cpu":"1","memory":"1Gi"}}]'
   ```
3. Check for volume mount issues:
   ```bash
   kubectl describe pod <pod-name> -n <namespace> | grep -A 10 "Volumes:"
   ```

## Deployment Issues

### Deployment Not Creating Pods

**Symptoms:**
- Deployment exists but no pods are created
- Replica count shows 0/N available

**Diagnostic Commands:**
```bash
# Check deployment status
kubectl describe deployment <deployment-name> -n <namespace>

# Check replica sets
kubectl get rs -n <namespace> -l app=<deployment-selector>
```

**Solutions:**

1. Check for admission controller issues:
   ```bash
   kubectl get validatingwebhookconfigurations
   kubectl get mutatingwebhookconfigurations
   ```

2. Verify pod spec is valid:
   ```bash
   kubectl apply --validate=true --dry-run=client -f deployment.yaml
   ```

3. Check for PodDisruptionBudget conflicts:
   ```bash
   kubectl get pdb -n <namespace>
   ```

### Deployment Stuck on Rolling Update

**Symptoms:**
- Deployment shows partial rollout
- New pods don't become ready

**Solutions:**

1. Check readiness probe failures:
   ```bash
   kubectl describe pod <new-pod-name> -n <namespace>
   ```

2. Adjust rollout strategy:
   ```bash
   kubectl patch deployment <deployment-name> -n <namespace> --type=json -p='[{"op":"replace","path":"/spec/strategy","value":{"type":"Recreate"}}]'
   ```

3. Rollback to previous version:
   ```bash
   kubectl rollout undo deployment/<deployment-name> -n <namespace>
   ```

## Service and Networking Issues

### Service Not Routing Traffic

**Symptoms:**
- Pods are running but service doesn't route traffic
- Endpoint connections time out

**Diagnostic Commands:**
```bash
# Check service and endpoints
kubectl describe service <service-name> -n <namespace>
kubectl get endpoints <service-name> -n <namespace>

# Verify label selectors
kubectl get pods -n <namespace> -l <service-selector>
```

**Solutions:**

1. Fix selector mismatch:
   ```bash
   # Update service selector to match pod labels
   kubectl patch service <service-name> -n <namespace> --type=json -p='[{"op":"replace","path":"/spec/selector","value":{"app":"<correct-label>"}}]'
   ```

2. Check pod readiness:
   ```bash
   kubectl get pods -n <namespace> -o wide
   ```

3. Test network connectivity:
   ```bash
   kubectl run test-$RANDOM --rm -it --image=busybox -n <namespace> -- wget -O- <service-name>:<port>
   ```

### Ingress Not Working

**Symptoms:**
- Service works internally but Ingress doesn't route external traffic
- Ingress controller logs show errors

**Solutions:**

1. Verify Ingress resource:
   ```bash
   kubectl describe ingress <ingress-name> -n <namespace>
   ```

2. Check Ingress controller logs:
   ```bash
   kubectl logs -n <ingress-controller-namespace> -l app=<ingress-controller> --tail=100
   ```

3. Verify TLS certificate if using HTTPS:
   ```bash
   kubectl get secret <tls-secret-name> -n <namespace>
   ```

## Volume and Storage Issues

### PersistentVolumeClaim Stuck in Pending

**Symptoms:**
- PVC remains in `Pending` state
- Pods requiring the PVC also stay in `Pending`

**Diagnostic Commands:**
```bash
# Check PVC status
kubectl describe pvc <pvc-name> -n <namespace>

# Check storage classes
kubectl get storageclass
```

**Solutions:**

1. Verify storage class exists and is default:
   ```bash
   kubectl get sc -o yaml
   ```

2. Check storage provisioner is running:
   ```bash
   kubectl get pods -n kube-system | grep provisioner
   ```

3. Create PV manually if using static provisioning:
   ```bash
   kubectl apply -f persistent-volume.yaml
   ```

### Volume Mount Failures

**Symptoms:**
- Pods fail to start with volume-related errors
- Events show "unable to mount volume"

**Solutions:**

1. Check volume types and paths:
   ```bash
   kubectl describe pod <pod-name> -n <namespace> | grep -A 15 "Volumes:"
   ```

2. Verify permissions on host paths:
   ```bash
   # For hostPath volumes on specific node
   kubectl debug node/<node-name> -it --image=ubuntu -- bash
   ls -la /path/on/host
   ```

3. Check if the PV was deleted:
   ```bash
   kubectl get pv | grep <pv-name>
   ```

## ConfigMap and Secret Issues

### ConfigMap or Secret Changes Not Reflected in Pods

**Symptoms:**
- Updated ConfigMap or Secret doesn't affect running pods
- Applications still use old configurations

**Solutions:**

1. Restart dependent pods:
   ```bash
   kubectl rollout restart deployment <deployment-name> -n <namespace>
   ```

2. Use latest Kubernetes best practices:
   ```bash
   # Add checksum annotation to trigger automatic restarts
   CHECKSUM=$(kubectl get cm <configmap-name> -n <namespace> -o yaml | sha256sum)
   kubectl patch deployment <deployment-name> -n <namespace> --type=json -p="[{\"op\":\"add\",\"path\":\"/spec/template/metadata/annotations/checksum\",\"value\":\"${CHECKSUM}\"}]"
   ```

3. Use ConfigMap subPath with caution - it won't auto-update

## Resource Quota and Limit Issues

### Namespace Resource Quota Exceeded

**Symptoms:**
- New resources can't be created
- Events show "exceeded quota" errors

**Diagnostic Commands:**
```bash
# Check resource quota usage
kubectl describe resourcequota -n <namespace>
```

**Solutions:**

1. Identify resource hogs:
   ```bash
   kubectl top pod -n <namespace>
   ```

2. Adjust quota limits:
   ```bash
   kubectl edit resourcequota <quota-name> -n <namespace>
   ```

3. Clean up unused resources:
   ```bash
   kubectl get all -n <namespace>
   ```

### LimitRange Conflicts

**Symptoms:**
- Pods fail validation
- Events show limit range errors

**Solutions:**

```bash
# Check limit range settings
kubectl get limitrange -n <namespace> -o yaml

# Adjust deployment resources
kubectl patch deployment <deployment-name> -n <namespace> --type=json -p='[{"op":"replace","path":"/spec/template/spec/containers/0/resources","value":{"requests":{"memory":"64Mi","cpu":"50m"},"limits":{"memory":"128Mi","cpu":"100m"}}}]'
```

## See Also

- [Working with Finalizers](./finalizers.md)
- [Namespace Cleanup](../namespace-management/cleanup.md)
- [ArgoCD Troubleshooting](../argocd/troubleshooting.md)

![hypermedia tech logo](../assets/images/hypermediatech-default.webp)