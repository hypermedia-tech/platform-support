# Kubernetes Tools Reference

This guide provides information on essential tools for Kubernetes cluster management, troubleshooting, and operations.

## Core Tools

### kubectl

The official Kubernetes command-line tool.

**Installation:**
```bash
# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# macOS
brew install kubectl
# or
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Windows
choco install kubernetes-cli
# or
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/windows/amd64/kubectl.exe"
```

**Key Features:**
- Manage Kubernetes resources
- View logs and execute commands in containers
- Forward ports and proxy connections
- Apply manifests and configurations

**Useful Extensions:**
- [kubectl aliases](https://github.com/ahmetb/kubectl-aliases)
- [kubectl plugins](https://krew.sigs.k8s.io/)

### Helm

Package manager for Kubernetes.

**Installation:**
```bash
# Linux/macOS
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Windows
choco install kubernetes-helm
```

**Basic Usage:**
```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Search charts
helm search repo bitnami

# Install chart
helm install my-release bitnami/nginx

# List releases
helm list

# Upgrade release
helm upgrade my-release bitnami/nginx --values custom-values.yaml

# Uninstall release
helm uninstall my-release
```

**Key Features:**
- Package and distribute applications
- Version control for deployments
- Template-based resource generation
- Release management

### k9s

Terminal-based UI for managing Kubernetes clusters.

**Installation:**
```bash
# Linux/macOS
brew install derailed/k9s/k9s

# Linux
curl -sS https://webinstall.dev/k9s | bash

# Windows
choco install k9s
```

**Key Features:**
- Interactive UI for cluster management
- Resource filtering and navigation
- Log viewing and container shell access
- Resource editing and management
- Customizable views and shortcuts

## Diagnostic Tools

### kubectl-debug

Debug running pods with ephemeral containers.

**Installation:**
```bash
kubectl krew install debug
```

**Usage:**
```bash
kubectl debug <pod-name> -it --image=nicolaka/netshoot
```

**Key Features:**
- Debug pods without modifying them
- Access running pods with debug tools
- Network and filesystem troubleshooting

### Stern

Multi-pod and container log tailing.

**Installation:**
```bash
# Linux/macOS
brew install stern

# Using go
go install github.com/stern/stern@latest
```

**Usage:**
```bash
# Tail logs from all pods with name containing "app"
stern app

# Tail logs for specific containers
stern app -c container1 -c container2
```

**Key Features:**
- Tail multiple pod logs simultaneously
- Color-coding for different pods/containers
- Regex filtering for pods and containers
- Timestamp and output formatting

### kubectx and kubens

Fast way to switch between contexts and namespaces.

**Installation:**
```bash
# Linux/macOS
brew install kubectx

# Using Krew
kubectl krew install ctx
kubectl krew install ns
```

**Usage:**
```bash
# Switch context
kubectx <context-name>

# Switch namespace
kubens <namespace>
```

**Key Features:**
- Fast context switching
- Namespace management
- Interactive selection

## Development Tools

### Skaffold

Continuous development for Kubernetes applications.

**Installation:**
```bash
# Linux/macOS
brew install skaffold

# Linux/Windows/macOS
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64
chmod +x skaffold
sudo mv skaffold /usr/local/bin
```

**Key Features:**
- Continuous build, push, and deploy
- File watching and hot reloading
- Multi-environment support
- Integration with CI/CD systems

### Kustomize

Template-free way to customize Kubernetes configurations.

**Installation:**
```bash
# Built into kubectl
kubectl kustomize

# Standalone
brew install kustomize
```

**Key Features:**
- Overlay-based configuration management
- Resource patching without templates
- Environment-specific configurations
- Integration with kubectl

## Monitoring and Visualization

### Prometheus and Grafana

Monitoring and visualization stack.

**Installation:**
```bash
# Using Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
helm install grafana grafana/grafana
```

**Key Features:**
- Time-series metrics collection and storage
- Alerting and notification
- Rich visualization dashboards
- Query language for metrics

### Lens

Kubernetes IDE with rich features.

**Installation:**
Download from [Lens website](https://k8slens.dev/)

**Key Features:**
- Graphical cluster management
- Resource visualization and metrics
- Terminal access and log viewing
- Built-in Prometheus integration

## Cluster Management

### kubeadm

Tool for cluster bootstrapping.

**Installation:**
```bash
apt-get update && apt-get install -y kubeadm
```

**Key Features:**
- Bootstrap Kubernetes clusters
- Upgrade Kubernetes components
- Add/remove nodes
- Generate certificates and tokens

### kops

Production-grade cluster installation, upgrade, management.

**Installation:**
```bash
# Linux/macOS
brew install kops
```

**Key Features:**
- Create production-ready clusters on AWS
- Multi-master high availability
- Infrastructure as code
- Rolling updates and upgrades

### Rancher

Complete container management platform.

**Installation:**
```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --create-namespace \
  --set hostname=rancher.example.com
```

**Key Features:**
- Multi-cluster management
- User management and RBAC
- App catalog and CI/CD
- Monitoring and logging integration

## GitOps Tools

### ArgoCD

Declarative GitOps continuous delivery tool.

**Installation:**
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**Key Features:**
- GitOps workflow
- Application state visualization
- Multi-cluster deployment
- SSO integration

### Flux

GitOps toolkit for Kubernetes.

**Installation:**
```bash
# Using Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash
flux bootstrap github \
  --owner=<username> \
  --repository=<repository> \
  --personal
```

**Key Features:**
- Automated deployment
- Helm and Kustomize integration
- Notification controller
- Multi-tenancy support

## Security Tools

### Trivy

Container vulnerability scanner.

**Installation:**
```bash
# Linux/macOS
brew install aquasecurity/trivy/trivy

# apt-based Linux
sudo apt-get install -y trivy
```

**Usage:**
```bash
trivy image nginx:latest
```

**Key Features:**
- Vulnerability scanning for containers
- Infrastructure as Code scanning
- Integration with CI/CD
- SBOM generation

### kube-bench

Kubernetes CIS benchmark testing.

**Installation:**
```bash
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml
```

**Key Features:**
- CIS benchmark compliance testing
- Security best practice verification
- Detailed remediation guidance
- CI/CD integration

## Network Tools

### kubeshark

API traffic analyzer for Kubernetes.

**Installation:**
```bash
# Linux/macOS
brew tap kubeshark/kubeshark
brew install kubeshark

# Script install
sh <(curl -Ls https://kubeshark.co/install)
```

**Key Features:**
- Real-time API traffic visualization
- Protocol-level inspection
- Traffic replay and testing
- Service map visualization

### Cilium

Advanced networking and security.

**Installation:**
```bash
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium --namespace kube-system
```

**Key Features:**
- eBPF-based networking
- Network policy enforcement
- Transparent encryption
- Network performance monitoring

## See Also

- [Kubectl Commands](./commands.md)
- [ArgoCD Operations](../argocd/index.md)
- [Resource Management](../resource-management/index.md)


![hypermedia tech logo](../assets/images/hypermediatech-default.webp)