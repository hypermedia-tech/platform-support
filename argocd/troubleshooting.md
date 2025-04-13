# ArgoCD Troubleshooting

This guide provides solutions for common ArgoCD issues and troubleshooting steps for maintaining a healthy ArgoCD deployment.

## Common Issues and Solutions

### Sync Failures

#### Issue: Application Won't Sync

**Symptoms:**
- Application shows "OutOfSync" status
- Sync operations fail or time out

**Troubleshooting Steps:**

1. Check for sync errors in the status:
   ```bash
   kubectl get application <app-name> -n argocd -o jsonpath='{.status.conditions}' | jq
   ```

2. Check resource constraints:
   ```bash
   kubectl top pod -n argocd
   ```

3. Examine application controller logs:
   ```bash
   kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller -c argocd-application-controller
   ```

**Solutions:**

- Force a refresh of the application:
  ```bash
  kubectl patch application <app-name> -n argocd --type=merge -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}'
  ```

- Restart the application controller:
  ```bash
  kubectl rollout restart deployment argocd-application-controller -n argocd
  ```

### Resource Issues

#### Issue: Resource is "Progressing" Forever

**Symptoms:**
- Resources stay in "Progressing" health status
- Deployments never reach ready state

**Troubleshooting Steps:**

1. Check the health status details:
   ```bash
   kubectl get application <app-name> -n argocd -o jsonpath='{.status.resources}' | jq 'map(select(.health.status=="Progressing"))'
   ```

2. Check the problematic resource directly:
   ```bash
   kubectl describe <resource-kind> <resource-name> -n <resource-namespace>
   ```

**Solutions:**

- Check for resource constraints in target namespace
- Verify the image exists and is accessible
- Check for pod errors using `kubectl describe pod`

#### Issue: Resources Out of Sync but No Changes

**Symptoms:**
- Application shows "OutOfSync" but Git comparison shows no differences

**Solutions:**

- Enable resource tracking via labels:
  ```bash
  kubectl patch application <app-name> -n argocd --type=merge -p '{"spec":{"syncPolicy":{"syncOptions":["CreateNamespace=true","RespectIgnoreDifferences=true"]}}}'
  ```

- Add ignore differences for fields that change frequently:
  ```bash
  kubectl patch application <app-name> -n argocd --type=merge -p '{"spec":{"ignoreDifferences":[{"group":"apps","kind":"Deployment","jsonPointers":["/spec/replicas"]}]}}'
  ```

### ApplicationSet Issues

#### Issue: ApplicationSet Not Generating Applications

**Symptoms:**
- No applications are created from an ApplicationSet
- Only some of the expected applications are created

**Troubleshooting Steps:**

1. Check ApplicationSet controller logs:
   ```bash
   kubectl logs -n argocd -l app.kubernetes.io/name=argocd-applicationset-controller
   ```

2. Verify the ApplicationSet spec:
   ```bash
   kubectl get applicationset <appset-name> -n argocd -o yaml
   ```

**Solutions:**

- Restart the ApplicationSet controller:
  ```bash
  kubectl rollout restart deployment argocd-applicationset-controller -n argocd
  ```

- Check if your generator (git, list, cluster, etc.) is correctly configured
- Verify Git repository access and credentials

### Authentication and Access Issues

#### Issue: Repository Connection Issues

**Symptoms:**
- Unable to connect to Git repository
- "connection refused" or SSL errors

**Troubleshooting Steps:**

1. Check repository secret:
   ```bash
   kubectl get secret <repo-secret> -n argocd -o yaml
   ```

2. Test connection using ArgoCD CLI:
   ```bash
   argocd repo list
   ```

**Solutions:**

- Update repository credentials
- Check network policies that might be blocking outbound connections
- Verify SSL certificates for private repositories

## ArgoCD Component Diagnostics

### Component Health Check

Run a quick diagnostic on all ArgoCD components:

```bash
kubectl get pods -n argocd
kubectl top pod -n argocd
kubectl describe statefulset,deployment -n argocd
```

### Fixing Stuck ArgoCD Components

If a component is in a bad state:

```bash
# Force restart the component
kubectl rollout restart deployment <component-name> -n argocd

# For example, to restart the API server:
kubectl rollout restart deployment argocd-server -n argocd
```

### Checking ArgoCD Server Health

```bash
# Check server status
kubectl exec -it $(kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | head -n 1) -n argocd -- argocd admin server health
```

## Recovering from Serious Issues

### Dealing with Corrupted Applications

1. Export the corrupted application:
   ```bash
   kubectl get application <app-name> -n argocd -o yaml > corrupted-app.yaml
   ```

2. Delete the application without cascade:
   ```bash
   kubectl delete application <app-name> -n argocd --cascade=false
   ```

3. Edit the exported YAML to fix issues, then reapply:
   ```bash
   kubectl apply -f fixed-app.yaml
   ```

### Recovering from Database Issues

If the ArgoCD Redis database is corrupted:

```bash
# Backup before attempting recovery
kubectl get all,secrets,configmaps,applications,appprojects -n argocd -o yaml > argocd-backup.yaml

# Delete and recreate Redis
kubectl delete pod -n argocd -l app.kubernetes.io/name=argocd-redis
```

## Performance Troubleshooting

### Addressing High CPU/Memory Usage

```bash
# Check resource usage
kubectl top pod -n argocd

# Scale up application controller if needed
kubectl scale deployment argocd-application-controller -n argocd --replicas=2

# Adjust application controller processing parallelism
kubectl edit configmap argocd-cmd-params-cm -n argocd
# Add --application-controller-operation-processors=10
```

### Optimizing for Large Clusters

For clusters with many applications:

```bash
# Set higher limits for the controller
kubectl patch deployment argocd-application-controller -n argocd --type=json -p='[{"op":"replace","path":"/spec/template/spec/containers/0/resources/limits","value":{"cpu":"2","memory":"4Gi"}}]'
```

## See Also

- [ArgoCD Application Management](./application-management.md)
- [Namespace Management](../namespace-management/index.md)
- [Working with Finalizers](../resource-management/finalizers.md)