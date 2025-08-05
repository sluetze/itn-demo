# Deployment Guide

## Overview

This guide provides step-by-step instructions for deploying the Red Hat ACM demo environments.

## Prerequisites

- Red Hat OpenShift clusters with ACM installed
- Git repository access
- Cluster admin permissions
- `oc` CLI tool configured

## Environment 1: Namespace-Based Deployment

### 1. Deploy Platform Team Resources

```bash
# Apply the namespace setup ApplicationSet (using Kustomize overlays)
oc apply -f environment-1-namespace-based/platform-team/applicationset-namespace-setup.yaml

# Apply governance policies
oc apply -f environment-1-namespace-based/platform-team/governance-policies.yaml

# Deploy the Web Portal application
oc apply -f environment-1-namespace-based/platform-team/applicationset-web-portal.yaml
```

### Configuration Variables
The namespace-based environment supports configurable variables through Kustomize:
- **Namespace Names**: `frontend-team-{dev,qa,prod}`
- **Route Hostnames**: `web-portal-{env}.apps.cluster.local`
- **Resource Limits**: Environment-specific CPU/memory quotas
- **Image Tags**: Environment-specific container image versions

### 2. Verify Deployment

```bash
# Check namespaces
oc get namespaces | grep frontend-team

# Check applications
oc get applications -n openshift-gitops | grep frontend-team

# Check policies
oc get policies -n default
```

### 3. Access the Application

- **Development**: https://web-portal-dev.apps.cluster.local
- **QA**: https://web-portal-qa.apps.cluster.local
- **Production**: https://web-portal.apps.cluster.local

## Environment 2: Cluster-Based Deployment

### 1. Label Your Clusters

```bash
# Apply cluster labels (modify cluster names and domains as needed)
oc apply -f environment-2-cluster-based/platform-team/cluster-labels.yaml
```

### Configuration Variables
The cluster-based environment supports configurable variables through cluster labels:
- **Cluster Names**: `development-cluster`, `qa-cluster`, `production-cluster`
- **Domains**: `apps.{env}-cluster.local` (configurable per cluster)
- **Route Hostnames**: Dynamically generated based on cluster domain
- **Resource Limits**: Cluster label-based CPU/memory allocation
- **Environment Labels**: Used for automatic ApplicationSet generation

### 2. Deploy Platform Team Resources

```bash
# Apply the cluster setup ApplicationSet (using Kustomize overlays)
oc apply -f environment-2-cluster-based/platform-team/applicationset-cluster-setup.yaml

# Apply governance policies
oc apply -f environment-2-cluster-based/platform-team/governance-policies.yaml

# Deploy the Web Portal application
oc apply -f environment-2-cluster-based/platform-team/applicationset-web-portal.yaml
```

### 3. Verify Multi-Cluster Deployment

```bash
# Check managed clusters
oc get managedclusters

# Check applications across clusters
oc get applications -n openshift-gitops | grep web-portal

# Check compliance
oc get policies -n default
```

## Application Architecture

The Web Portal application includes:

- **Frontend**: NGINX-based web server
- **Configuration**: Environment-specific ConfigMaps
- **Networking**: OpenShift Routes with TLS
- **Scaling**: HPA for production environment
- **Security**: Pod Security Standards, Network Policies

## Customization

### Modifying Application

1. Update the application manifests in `application/base/`
2. Customize environment-specific settings in `application/overlays/`
3. Commit changes to trigger GitOps deployment

### Adding New Environments

1. Create new overlay in `application/overlays/`
2. Update ApplicationSet generators
3. Apply configuration changes

## Troubleshooting

### Common Issues

1. **Application not syncing**
   ```bash
   oc get applications -n openshift-gitops
   oc describe application <app-name> -n openshift-gitops
   ```

2. **Policy violations**
   ```bash
   oc get policies -n default
   oc describe policy <policy-name> -n default
   ```

3. **Resource quotas exceeded**
   ```bash
   oc describe resourcequota -n <namespace>
   ```

### Log Analysis

```bash
# Application logs
oc logs deployment/web-portal -n <namespace>

# ArgoCD logs
oc logs deployment/argocd-application-controller -n openshift-gitops

# ACM logs
oc logs deployment/multicluster-operators-application -n open-cluster-management
```