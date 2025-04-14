# Kubernetes Commands Reference

This reference guide provides commonly used kubectl commands organized by category, with examples and explanations.

## Cluster Information

### Cluster Status

```bash
# Get cluster info
kubectl cluster-info

# Check component status
kubectl get componentstatuses

# Get nodes with details
kubectl get nodes -o wide

# Node details
kubectl describe node <node-name>
```

### API Resources

```bash
# List all API resources
kubectl api-resources

# List API resources with specific verbs
kubectl api-resources --verbs=list,get

# Get API versions
kubectl api-versions

# Explain resource
kubectl explain <resource>
kubectl explain pod.spec.containers
```

## Resource Management

### Basic Operations

```bash
# Create resources
kubectl create -f <file-or-dir>

# Apply manifests (create or update)
kubectl apply -f <file-or-dir>

# Delete resources
kubectl delete -f <file-or-dir>

# Scale resources
kubectl scale deployment <name> --replicas=<count>

# Edit resources
kubectl edit <resource> <name>

# Replace resource
kubectl replace -f <file>

# Patch resource
kubectl patch <resource> <name> --type=json -p='[{"op":"replace","path":"/spec/replicas","value":3}]'
```

### Viewing Resources

```bash
# Get resources of a specific type
kubectl get <resource-type>

# Get all resources
kubectl get all -n <namespace>

# Get resources with more details
kubectl get <resource-type> -o wide

# Watch for changes
kubectl get <resource-type> -w

# Format output as YAML
kubectl get <resource-type> <name> -o yaml

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

# Sorting
kubectl get pods --sort-by=.metadata.creationTimestamp

# Filtering with field selectors
kubectl get pods --field-selector=status.phase=Running

# Filtering with label selectors
kubectl get pods -l app=nginx,tier=frontend
```

### Resource Details

```bash
# Describe resources
kubectl describe <resource-type> <name>

# Get specific fields using jsonpath
kubectl get <resource> <name> -o jsonpath='{.spec.replicas}'

# View resource hierarchy
kubectl get deployment <name> -o jsonpath='{.metadata.ownerReferences}'
```

## Pods and Containers

### Pod Operations

```bash
# Create pod from image
kubectl run <name> --image=<image>

# Get pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl logs -f <pod-name> # Follow logs
kubectl logs <pod-name> --tail=100 # Last 100 lines
kubectl logs <pod-name> --since=1h # Logs from last hour

# Execute commands in containers
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec <pod-name> -- <command>

# Copy files to/from pods
kubectl cp <file> <pod-name>:/<path>
kubectl cp <pod-name>:/<path> <local-path>

# Port forwarding
kubectl port-forward <pod-name> <local-port>:<pod-port>

# Get pod resource usage
kubectl top pod <pod-name>
```

### Pod Troubleshooting

```bash
# Force delete pod
kubectl delete pod <pod-name> --force --grace-period=0

# Check pod events
kubectl get events --field-selector involvedObject.name=<pod-name>

# Get previous container logs
kubectl logs <pod-name> --previous

# Debug with ephemeral container (K8s 1.23+)
kubectl debug <pod-name> -it --image=busybox
```

## Deployment Management

```bash
# Create deployment
kubectl create deployment <name> --image=<image>

# Update deployment image
kubectl set image deployment/<name> <container>=<new-image>

# View rollout status
kubectl rollout status deployment/<name>

# Rollout history
kubectl rollout history deployment/<name>

# Rollback to previous version
kubectl rollout undo deployment/<name>
kubectl rollout undo deployment/<name> --to-revision=<revision>

# Pause/Resume rollout
kubectl rollout pause deployment/<name>
kubectl rollout resume deployment/<name>

# Restart deployment
kubectl rollout restart deployment/<name>
```

## Service Management

```bash
# Create service
kubectl expose deployment <name> --port=<port> --target-port=<target-port>

# Get endpoints
kubectl get endpoints <service-name>

# Test service DNS
kubectl run -it --rm debug --image=busybox -- nslookup <service-name>

# Test service connection
kubectl run -it --rm debug --image=busybox -- wget -O- <service-name>:<port>
```

## Namespace Operations

```bash
# Create namespace
kubectl create namespace <name>

# Set default namespace
kubectl config set-context --current --namespace=<namespace>

# List resources across all namespaces
kubectl get <resource> --all-namespaces

# Delete namespace and all contents
kubectl delete namespace <name>

# Get resources that aren't namespaced 
kubectl api-resources --namespaced=false
```

## Config and Context

```bash
# View kubeconfig
kubectl config view

# Get contexts
kubectl config get-contexts

# Switch context
kubectl config use-context <context-name>

# Rename context
kubectl config rename-context <old-name> <new-name>

# Delete context
kubectl config delete-context <context-name>

# Set cluster/user/namespace for a context
kubectl config set-context <context-name> --cluster=<cluster-name> --user=<user-name> --namespace=<namespace>
```

## Resource Quotas and Limits

```bash
# View resource quotas
kubectl get resourcequota -n <namespace>

# Describe quota usage
kubectl describe resourcequota <quota-name> -n <namespace>

# Get limit ranges
kubectl get limitrange -n <namespace>
```

## RBAC Management

```bash
# Create service account
kubectl create serviceaccount <name>

# Create role
kubectl create role <name> --verb=get,list,watch --resource=pods

# Create rolebinding
kubectl create rolebinding <name> --role=<role-name> --serviceaccount=<namespace>:<sa-name>

# Check permissions
kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<namespace>:<serviceaccount>
```

## ConfigMaps and Secrets

```bash
# Create configmap
kubectl create configmap <name> --from-file=<path> --from-literal=key=value

# Create secret
kubectl create secret generic <name> --from-file=<path> --from-literal=key=value

# Decode secret
kubectl get secret <name> -o jsonpath='{.data.<key>}' | base64 --decode
```

## Batch Operations

```bash
# Delete all pods in a namespace
kubectl delete pods --all -n <namespace>

# Apply manifests recursively
kubectl apply -f <directory> --recursive

# Get all resources with a specific label
kubectl get all -l app=<app-label>

# Delete resources based on label
kubectl delete all -l app=<app-label>
```

## Cluster Administration

```bash
# Drain node for maintenance
kubectl drain <node-name> --ignore-daemonsets

# Cordon/uncordon node
kubectl cordon <node-name>    # Mark node as unschedulable
kubectl uncordon <node-name>  # Mark node as schedulable

# Taint nodes
kubectl taint nodes <node-name> key=value:effect

# Manage static pods
# Edit files in /etc/kubernetes/manifests/ on the node
```

## See Also

- [Kubernetes Tools](./tools.md)
- [Resource Management](../resource-management/index.md)
- [ArgoCD Operations](../argocd/index.md)

![hypermedia tech logo](../assets/images/hypermediatech-default.webp)