# Kubernetes Descheduler Configuration

This repository contains working descheduler configurations for Kubernetes.

## Quick Start

Since your cluster **does not have DeschedulerPolicy CRD**, use the ConfigMap-based approach:

```bash
# Deploy the descheduler as a CronJob
kubectl apply -f descheduler-cronjob.yaml

# Check status
./manage-descheduler.sh status

# Trigger manually for testing
./manage-descheduler.sh trigger

# View logs
./manage-descheduler.sh logs
```

The descheduler can be run as a Job, CronJob, or Deployment inside of a k8s cluster. It has the advantage of being able to be run multiple times without needing user intervention. The descheduler pod is run as a critical pod in the kube-system namespace to avoid being evicted by itself or by the kubelet.

Run As A Job

kubectl create -f kubernetes/base/rbac.yaml
kubectl create -f kubernetes/base/configmap.yaml
kubectl create -f kubernetes/job/job.yaml

Run As A CronJob
kubectl create -f kubernetes/base/rbac.yaml
kubectl create -f kubernetes/base/configmap.yaml
kubectl create -f kubernetes/cronjob/cronjob.yaml

Run As A Deployment
kubectl create -f kubernetes/base/rbac.yaml
kubectl create -f kubernetes/base/configmap.yaml
kubectl create -f kubernetes/deployment/deployment.yaml




## Files Structure

### ✅ **Main Files (Use These)**
- `descheduler-cronjob.yaml` - **Main CronJob configuration (ConfigMap-based)**
- `manage-descheduler.sh` - **Management script for the CronJob**
- `check-crd.sh` - Script to check for CRDs and existing resources

### 📝 **Helm Values Files**
- `values-cronjob-example.yaml` - Production-safe Helm values for CronJob
- `values-deployment-namespace-protection.yaml` - Deployment with namespace protection
- `values-cronjob.yaml` - Original comprehensive Helm values

### 📖 **Reference Files**
- `deployment.yaml` - Sample deployment (not descheduler related)
- `deschedule.yaml` - Sample policy file for reference
- `pod.yaml` - Sample pod configuration

### ❌ **Don't Use**
- `descheduler-cronjob-crd.yaml` - Requires CRD (not available in your cluster)

## Configuration Details

The main `descheduler-cronjob.yaml` includes:

- **Schedule**: Runs every 30 minutes
- **Policies**: 
  - RemoveDuplicates (with namespace protection)
  - LowNodeUtilization (balanced resource usage)
  - RemovePodsHavingTooManyRestarts (restart threshold: 3)
- **Protection**: Excludes kube-system, kube-public, kube-node-lease
- **Security**: Non-root, read-only filesystem, minimal privileges

## Monitoring

```bash
# Check CronJob status
kubectl get cronjob -n kube-system descheduler-cronjob

# View recent jobs
kubectl get jobs -n kube-system -l app=descheduler

# View pod events
kubectl describe cronjob -n kube-system descheduler-cronjob
```

## Troubleshooting

1. **CRD Not Found Error**: Use `descheduler-cronjob.yaml` (ConfigMap-based), not the CRD version
2. **Permission Errors**: Check if RBAC resources were created properly
3. **No Pods Evicted**: Check if your cluster has pods that match the eviction criteria

## Management Commands

```bash
# Make script executable
chmod +x manage-descheduler.sh

# Check environment
./manage-descheduler.sh check

# Deploy
./manage-descheduler.sh deploy

# Monitor
./manage-descheduler.sh status
./manage-descheduler.sh logs

# Control
./manage-descheduler.sh suspend
./manage-descheduler.sh resume
```


https://hwchiu.medium.com/exploring-kubernetes-descheduler-0b69903ff109


https://github.com/kubernetes-sigs/descheduler/issues/1561