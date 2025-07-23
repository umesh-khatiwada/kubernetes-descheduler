# Kubernetes Descheduler Configuration

This repository contains working descheduler configurations for Kubernetes w### 🚀 **Main Deployment Files**
- `kubernetes/base/` - Core RBAC and ConfigMap resources
- `kubernetes/cronjob/` - **Recommended**: Periodic descheduling
- `kubernetes/job/` - One-time descheduling operations  
- `kubernetes/deployment/` - Continuous descheduling

### 📚 **Example Configurations**
- `examples/policy.yaml` - Complete descheduler policy example
- `examples/*.yml` - Specific policy examples (node utilization, pod lifetime, etc.)
- `examples/cronjob/` - CronJob-specific examples
- `examples/deployment/` - Deployment examples

### 🎯 **Helm Charts**deployment options and example policies.

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Deployment Options](#deployment-options)
- [Repository Structure](#repository-structure)
- [Current Configuration](#current-configuration)
- [Monitoring & Management](#monitoring--management)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)
- [Production Considerations](#production-considerations)
- [Resources & References](#resources--references)
- [Contributing](#contributing)

## Overview

The Kubernetes Descheduler helps improve cluster resource utilization by identifying and evicting pods based on configurable policies. This repository provides ready-to-use configurations for different deployment methods.

## Quick Start

Choose your deployment method based on your needs:

### Option 1: Using Kustomize (Recommended)
```bash
# Deploy as a CronJob (runs periodically)
kubectl apply -k kubernetes/cronjob/

# Or deploy as a one-time Job
kubectl apply -k kubernetes/job/

# Or deploy as a continuous Deployment
kubectl apply -k kubernetes/deployment/

# Verify deployment
kubectl get cronjob,job,deployment -n kube-system -l app=descheduler
```

### Option 2: Manual Deployment
```bash
# Deploy RBAC and ConfigMap first
kubectl apply -f kubernetes/base/rbac.yaml
kubectl apply -f kubernetes/base/configmap.yaml

# Then choose your deployment type:
kubectl apply -f kubernetes/cronjob/cronjob.yaml
# OR
kubectl apply -f kubernetes/job/job.yaml
# OR 
kubectl apply -f kubernetes/deployment/deployment.yaml
```

### Option 3: Using Official Upstream Kustomize
If you prefer to use the latest official descheduler images directly:

```bash
# Run as a Job
kustomize build 'github.com/kubernetes-sigs/descheduler/kubernetes/job?ref=release-1.33' | kubectl apply -f -

# Run as a CronJob
kustomize build 'github.com/kubernetes-sigs/descheduler/kubernetes/cronjob?ref=release-1.33' | kubectl apply -f -

# Run as a Deployment
kustomize build 'github.com/kubernetes-sigs/descheduler/kubernetes/deployment?ref=release-1.33' | kubectl apply -f -
```

> **Note**: The upstream version uses default policies. Use the local configurations in this repository for customized policies and examples.

## Deployment Options

The descheduler can be run as a Job, CronJob, or Deployment inside a Kubernetes cluster. It runs as a critical pod in the kube-system namespace to avoid being evicted.

### CronJob (Recommended for Production)
- **Use case**: Periodic optimization runs
- **Schedule**: Configurable (default: every minute for testing)
- **Location**: `kubernetes/cronjob/`

### Job
- **Use case**: One-time descheduling operations
- **Location**: `kubernetes/job/`

### Deployment 
- **Use case**: Continuous monitoring and descheduling
- **Location**: `kubernetes/deployment/`

## Repository Structure

```
├── kubernetes/                    # Main deployment configurations
│   ├── base/                     # Shared base resources
│   │   ├── configmap.yaml        # Descheduler policy configuration
│   │   ├── rbac.yaml            # Role-based access control
│   │   └── kustomization.yaml   # Kustomize base configuration
│   ├── cronjob/                 # CronJob deployment
│   │   ├── cronjob.yaml         # CronJob resource definition
│   │   └── kustomization.yaml   # Kustomize overlay
│   ├── job/                     # One-time Job deployment
│   │   ├── job.yaml            # Job resource definition
│   │   └── kustomization.yaml  # Kustomize overlay
│   └── deployment/              # Continuous deployment
│       ├── deployment.yaml     # Deployment resource definition
│       └── kustomization.yaml  # Kustomize overlay
├── examples/                    # Example configurations and policies
│   ├── policy.yaml             # Complete policy example
│   ├── policy/                 # Policy configurations
│   ├── cronjob/               # CronJob examples
│   ├── deployment/            # Deployment examples
│   ├── *.yml                  # Various policy examples
│   └── *.yaml                 # Additional examples
├── helm/                       # Helm chart configurations
│   ├── values-cronjob.yaml    # Helm values for CronJob
│   └── values-unified.yaml    # Unified Helm values
└── assests/                    # Documentation images
    └── *.png                   # Architecture diagrams
```

### 🚀 **Main Deployment Files**
- `kubernetes/base/` - Core RBAC and ConfigMap resources
- `kubernetes/cronjob/` - **Recommended**: Periodic descheduling
- `kubernetes/job/` - One-time descheduling operations  
- `kubernetes/deployment/` - Continuous descheduling

### � **Example Configurations**
- `examples/policy.yaml` - Complete descheduler policy example
- `examples/*.yml` - Specific policy examples (node utilization, pod lifetime, etc.)
- `examples/cronjob/` - CronJob-specific examples
- `examples/deployment/` - Deployment examples

### 🎯 **Helm Charts**
- `helm/values-cronjob.yaml` - Production-ready CronJob values
- `helm/values-unified.yaml` - Comprehensive Helm configuration

## Current Configuration

The default configuration in `kubernetes/base/configmap.yaml` includes:

### **Enabled Policies**
- **RemovePodsViolatingInterPodAntiAffinity** - Removes pods violating anti-affinity rules
- **RemoveDuplicates** - Removes duplicate pods (multiple pods of same ReplicaSet on same node)  
- **LowNodeUtilization** - Balances workload across nodes
  - **Thresholds**: CPU: 20%, Memory: 20%, Pods: 20%
  - **Target Thresholds**: CPU: 50%, Memory: 50%, Pods: 50%

### **Schedule & Security**
- **CronJob Schedule**: `* * * * *` (every minute - adjust for production)
- **Security**: Non-root user, read-only filesystem, dropped capabilities
- **Priority**: System-critical priority class
- **Resources**: 500m CPU request, 256Mi memory request

## Monitoring & Management

### Check Status
```bash
# CronJob status
kubectl get cronjob -n kube-system descheduler-cronjob

# Recent jobs and pods
kubectl get jobs -n kube-system -l app=descheduler
kubectl get pods -n kube-system -l app=descheduler

# Job details and events
kubectl describe cronjob -n kube-system descheduler-cronjob
```

### View Logs
```bash
# Get the latest pod name
LATEST_POD=$(kubectl get pods -n kube-system -l app=descheduler --sort-by=.metadata.creationTimestamp -o jsonpath='{.items[-1].metadata.name}')

# View logs
kubectl logs -n kube-system $LATEST_POD

# Follow logs for running pod
kubectl logs -n kube-system $LATEST_POD -f
```

### Manual Trigger (CronJob)
```bash
# Create a manual job from the cronjob
kubectl create job -n kube-system manual-descheduler-$(date +%s) --from=cronjob/descheduler-cronjob
```

## Customization

### Modify Policies
Edit the ConfigMap to customize descheduling behavior:

```bash
kubectl edit configmap -n kube-system descheduler-policy-configmap
```

### Change Schedule (CronJob)
Update the CronJob schedule:

```bash
kubectl patch cronjob -n kube-system descheduler-cronjob -p '{"spec":{"schedule":"0 */2 * * *"}}'  # Every 2 hours
```

### Available Policy Examples
Check the `examples/` directory for additional policy configurations:
- `high-node-utilization.yml` - Target high resource utilization
- `low-node-utilization.yml` - Balance low utilization
- `node-affinity.yml` - Handle node affinity violations
- `pod-life-time.yml` - Remove long-running pods
- `too-many-restarts.yml` - Remove pods with excessive restarts
- `topology-spread-constraint.yaml` - Handle topology spread violations

## Troubleshooting

### Common Issues

1. **No Pods Evicted**
   - Check if workloads match the descheduling criteria
   - Verify eviction policies in the ConfigMap
   - Ensure pods don't have `descheduler.alpha.kubernetes.io/evict: "false"` annotation

2. **Permission Errors**
   ```bash
   # Verify RBAC is applied
   kubectl get clusterrole descheduler-cluster-role
   kubectl get clusterrolebinding descheduler-cluster-role-binding
   ```

3. **Pod Scheduling Issues**
   ```bash
   # Check pod events
   kubectl describe pod -n kube-system -l app=descheduler
   ```

4. **High Resource Usage**
   - Adjust CPU/memory requests in the deployment
   - Consider reducing descheduling frequency

### Debug Mode
Enable verbose logging by modifying the args in your deployment:
```yaml
args:
  - "--policy-config-file"
  - "/policy-dir/policy.yaml"
  - "--v"
  - "4"  # Increase verbosity (0-4)
```

## Production Considerations

### Security
- All deployments use non-root users and read-only filesystems
- Minimal required permissions via RBAC
- System-critical priority to prevent self-eviction

### Resource Management
- Configure appropriate resource requests/limits
- Monitor descheduler resource usage
- Consider cluster size when setting schedules

### Scheduling Best Practices
- **Development**: Every few minutes for testing
- **Staging**: Every 30 minutes to 1 hour  
- **Production**: Every 2-6 hours (depends on cluster dynamics)

### Namespace Protection
Consider excluding critical namespaces by modifying the policy:
```yaml
nodeSelector: "node.kubernetes.io/exclude-from-external-load-balancers!=true"
namespaces:
  exclude:
    - "kube-system"
    - "kube-public" 
    - "kube-node-lease"
    - "your-critical-namespace"
```

## Resources & References

- [Official Descheduler Documentation](https://github.com/kubernetes-sigs/descheduler)
- [Descheduler Policies Guide](https://github.com/kubernetes-sigs/descheduler/blob/master/docs/user-guide.md)
- [Exploring Kubernetes Descheduler](https://hwchiu.medium.com/exploring-kubernetes-descheduler-0b69903ff109)
- [Descheduler Policy Examples](https://github.com/kubernetes-sigs/descheduler/tree/master/examples)

## Contributing

When contributing to this repository:
1. Test configurations in a development cluster first
2. Validate YAML syntax and Kubernetes resource definitions
3. Update examples and documentation accordingly  
4. Follow Kubernetes best practices for security and resource management