# Descheduler Helm Charts

This directory contains Helm values files for deploying the Kubernetes Descheduler using the official [descheduler Helm chart](https://github.com/kubernetes-sigs/descheduler/tree/master/charts/descheduler).

## Overview

The Kubernetes Descheduler improves cluster resource utilization by identifying and evicting pods based on configurable policies. These Helm values provide production-ready configurations for different deployment scenarios.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- Sufficient RBAC permissions to create ClusterRoles and ClusterRoleBindings

## Quick Start

### 1. Add the Official Descheduler Helm Repository

```bash
helm repo add descheduler https://kubernetes-sigs.github.io/descheduler/
helm repo update
```

### 2. Deploy Using Provided Values

```bash
# Option 1: CronJob deployment (recommended for production)
helm install descheduler descheduler/descheduler \
  --namespace kube-system \
  --values values-cronjob.yaml

# Option 2: Unified deployment with comprehensive configuration
helm install descheduler descheduler/descheduler \
  --namespace kube-system \
  --values values-unified.yaml
```

### 3. Verify Deployment

```bash
# Check CronJob deployment
kubectl get cronjob -n kube-system descheduler

# Check Deployment (if using unified values with Deployment mode)
kubectl get deployment -n kube-system descheduler

# View recent jobs and pods
kubectl get jobs,pods -n kube-system -l app.kubernetes.io/name=descheduler
```

## Available Values Files

### 📋 `values-cronjob.yaml`
**Recommended for Production**

- **Deployment Type**: CronJob
- **Schedule**: Every 2 minutes (configurable)
- **Policies**: Comprehensive set including:
  - RemoveDuplicates
  - RemovePodsHavingTooManyRestarts (threshold: 100)
  - RemovePodsViolatingNodeAffinity
  - RemovePodsViolatingNodeTaints
  - RemovePodsViolatingInterPodAntiAffinity
  - RemovePodsViolatingTopologySpreadConstraint
  - LowNodeUtilization (CPU: 20%→50%, Memory: 20%→50%)
- **Security**: Non-root, read-only filesystem, system-critical priority
- **Resources**: 500m CPU request, 256Mi memory

### 🔧 `values-unified.yaml`
**Comprehensive Configuration for Advanced Users**

- **Deployment Types**: Supports both Deployment and CronJob modes
- **Leader Election**: Enabled for Deployment mode (3 replicas)
- **Flexible Resources**: Production and development resource profiles
- **Environment-Specific**: Comments for easy environment switching

## Configuration Options

### Deployment Types

#### CronJob Mode (Recommended)
```yaml
kind: CronJob
schedule: "*/30 * * * *"  # Every 30 minutes
suspend: false
```

#### Deployment Mode (Continuous)
```yaml
kind: Deployment
replicas: 3
leaderElection:
  enabled: true
deschedulingInterval: 5m
```

### Security Configuration

```yaml
securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop: [ALL]
  privileged: false
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 1000

priorityClassName: system-cluster-critical
```

### Policy Customization

The values files include comprehensive descheduling policies. Key policies:

```yaml
deschedulerPolicy:
  profiles:
    - name: default
      pluginConfig:
        - name: LowNodeUtilization
          args:
            thresholds:
              cpu: 20    # Nodes below 20% are underutilized
              memory: 20
              pods: 20
            targetThresholds:
              cpu: 50    # Target 50% utilization
              memory: 50
              pods: 50
```

## Common Customizations

### 1. Change Schedule (CronJob)

```bash
# Every hour
helm upgrade descheduler descheduler/descheduler \
  --namespace kube-system \
  --values values-cronjob.yaml \
  --set schedule="0 * * * *"

# Every 6 hours (production recommendation)
helm upgrade descheduler descheduler/descheduler \
  --namespace kube-system \
  --values values-cronjob.yaml \
  --set schedule="0 */6 * * *"
```

### 2. Adjust Resource Limits

```bash
helm upgrade descheduler descheduler/descheduler \
  --namespace kube-system \
  --values values-cronjob.yaml \
  --set resources.limits.cpu=1000m \
  --set resources.limits.memory=512Mi
```

### 3. Enable/Disable Policies

```bash
# Disable a specific policy
helm upgrade descheduler descheduler/descheduler \
  --namespace kube-system \
  --values values-cronjob.yaml \
  --set deschedulerPolicy.profiles[0].plugins.balance.enabled[0]=null
```

### 4. Environment-Specific Deployments

```bash
# Development environment
helm install descheduler-dev descheduler/descheduler \
  --namespace kube-system \
  --values values-unified.yaml \
  --set schedule="*/5 * * * *" \
  --set resources.requests.cpu=200m

# Production environment  
helm install descheduler-prod descheduler/descheduler \
  --namespace kube-system \
  --values values-unified.yaml \
  --set schedule="0 */2 * * *" \
  --set resources.limits.cpu=1000m
```

## Monitoring and Troubleshooting

### Check Status

```bash
# CronJob status
kubectl describe cronjob -n kube-system descheduler

# Recent executions
kubectl get jobs -n kube-system -l app.kubernetes.io/name=descheduler --sort-by=.metadata.creationTimestamp

# Pod logs
kubectl logs -n kube-system -l app.kubernetes.io/name=descheduler --tail=100
```

### Manual Trigger (CronJob)

```bash
# Create a manual job from the cronjob
kubectl create job -n kube-system manual-descheduler-$(date +%s) \
  --from=cronjob/descheduler
```

### Debug Mode

```bash
# Enable verbose logging
helm upgrade descheduler descheduler/descheduler \
  --namespace kube-system \
  --values values-cronjob.yaml \
  --set cmdOptions.v=4
```

## Production Best Practices

### 1. Scheduling Recommendations
- **Small clusters** (< 50 nodes): Every 1-2 hours
- **Medium clusters** (50-200 nodes): Every 2-4 hours  
- **Large clusters** (> 200 nodes): Every 4-6 hours

### 2. Resource Sizing
- **CPU**: Start with 500m, monitor and adjust
- **Memory**: 256Mi for small clusters, 512Mi+ for large clusters
- **Replicas**: Use 3+ replicas for Deployment mode with leader election

### 3. Security Considerations
- Always use non-root security contexts (included in values)
- Set appropriate resource limits to prevent resource exhaustion
- Use system-critical priority class to prevent self-eviction

### 4. Policy Tuning
- Start with conservative thresholds and adjust based on cluster behavior
- Monitor eviction events and adjust policies accordingly
- Consider excluding critical namespaces from certain policies

## Migration from kubectl

If migrating from kubectl-based deployments:

```bash
# Remove existing resources
kubectl delete -k ../kubernetes/cronjob/

# Install using Helm
helm install descheduler descheduler/descheduler \
  --namespace kube-system \
  --values values-cronjob.yaml
```

## Uninstalling

```bash
# Uninstall the Helm release
helm uninstall descheduler --namespace kube-system

# Verify cleanup
kubectl get all -n kube-system -l app.kubernetes.io/name=descheduler
```

## Contributing

When modifying values files:
1. Test in a development environment first
2. Validate against the official descheduler chart schema
3. Update this README with any new configuration options
4. Consider backward compatibility for existing deployments

## Resources

- [Official Descheduler Chart](https://github.com/kubernetes-sigs/descheduler/tree/master/charts/descheduler)
- [Descheduler Documentation](https://github.com/kubernetes-sigs/descheduler)
- [Policy Configuration Guide](https://github.com/kubernetes-sigs/descheduler/blob/master/docs/user-guide.md)
- [Helm Documentation](https://helm.sh/docs/)
