# Red Hat ACM Demo: Empowering Application Teams with Self-Service Platform

**Target Audience**: Platform Engineers, DevOps Leads, and IT Management
**Duration**: 25-30 minutes
**Goal**: Demonstrate how Red Hat Advanced Cluster Management (ACM) enables secure, scalable self-service for application teams while maintaining platform governance.

## Demo Scenario

Our demo showcases the **Frontend Team** deploying their **Web Portal** application across development, QA, and production environments using two different infrastructure approaches:

1. **Environment 1**: Single cluster with namespace-based environment separation (`environment-1-namespace-based/`)
2. **Environment 2**: Multi-cluster setup with environment-specific clusters (`environment-2-cluster-based/`)

Both environments demonstrate:
- Centralized governance and policy enforcement
- Automated resource provisioning and RBAC
- GitOps-based application deployment
- Environment-specific configurations and scaling

## 1. Introduction & Setup (5 minutes)

### Opening Statement
*"Good morning, everyone. Today we'll explore how Red Hat Advanced Cluster Management solves a critical challenge: enabling application team agility while maintaining enterprise governance and security standards.*

*Our scenario involves the Frontend Team deploying their Web Portal application. We'll demonstrate two infrastructure patterns that organizations commonly use, showing how ACM adapts to different operational models while providing consistent governance."*

### Platform Team Goals
- **Centralized Governance**: Consistent policy enforcement across all environments
- **Resource Isolation**: Dedicated namespaces with appropriate quotas and limits
- **Self-Service Capability**: Empowering teams without compromising security
- **Unified Visibility**: Single dashboard for monitoring and compliance

*Show the ACM UI with both environment setups*

## 2. Platform Team Foundation (10 minutes)

### Environment 1: Namespace-Based Setup

*"Let's start with Environment 1, where a single cluster hosts multiple environments using namespace separation."*

#### ApplicationSet for Namespace Provisioning
*Show: `environment-1-namespace-based/platform-team/applicationset-namespace-setup.yaml`*

**Key Features:**
- **List Generator**: Defines dev, qa, prod with specific resource allocations
- **Helm Parameters**: Environment-specific CPU, memory, and storage limits
- **Namespace Creation**: Automated provisioning with proper labeling

```bash
oc apply -f environment-1-namespace-based/platform-team/applicationset-namespace-setup.yaml
```

#### Governance Policies
*Show: `environment-1-namespace-based/platform-team/governance-policies.yaml`*

**Demonstrate:**
- Security context enforcement
- Network policy isolation
- Resource limit validation

### Environment 2: Cluster-Based Setup

*"Now let's examine Environment 2, where each environment has its dedicated cluster."*

#### Cluster Labels Configuration
*Show: `environment-2-cluster-based/platform-team/cluster-labels.yaml`*

#### Multi-Cluster ApplicationSet
*Show: `environment-2-cluster-based/platform-team/applicationset-cluster-setup.yaml`*

**Key Differences:**
- **Cluster Generator**: Automatically discovers labeled clusters
- **Dynamic Resource Allocation**: Uses cluster labels for configurations
- **Cross-Cluster Governance**: Consistent policies across multiple clusters

```bash
oc apply -f environment-2-cluster-based/platform-team/applicationset-cluster-setup.yaml
```

*"ACM is now automatically creating the following resources:*
- *Namespaces with appropriate labels and quotas*
- *RBAC configurations for team access*
- *Security policies and network isolation*
- *Environment-specific resource limits"*

## 3. Application Team Self-Service (8 minutes)

### Application Architecture Overview
*"The Frontend Team's Web Portal is designed as a cloud-native application:"*

- **No persistent storage** - Stateless design
- **Environment-specific configurations** - Dev, QA, Prod variants
- **Web-accessible frontend** - NGINX-based with health checks
- **Production scaling** - HPA for automatic scaling

### GitOps Deployment Workflow

#### Base Application Structure
*Show: `application/base/deployment.yaml`*

**Security Features:**
- Non-root user execution
- Security context constraints
- Comprehensive health checks
- Resource limits and requests

#### Environment-Specific Overlays

**Development Configuration:**
*Show: `application/overlays/dev/kustomization.yaml`*
- Single replica, debug mode enabled, relaxed resource limits

**Production Configuration:**
*Show: `application/overlays/prod/kustomization.yaml`*
- Multi-replica deployment, HPA, production security hardening

### Deployment Demonstration

#### Environment 1 Deployment
```bash
oc apply -f environment-1-namespace-based/platform-team/applicationset-web-portal.yaml
```

#### Environment 2 Deployment
```bash
oc apply -f environment-2-cluster-based/platform-team/applicationset-web-portal.yaml
```

*Show ArgoCD dashboard with applications deploying across environments*

### Live Application Updates
*"Let's simulate the Frontend Team making a change:"*

1. **Change application image tag** from `v1.0` to `v1.1`
2. **Commit to Git repository**
3. **Watch automated synchronization** across all environments

*"ACM has detected the change in our Git repository. It's now automatically pulling the new manifests and applying them to our environments. The Frontend Team has full control over their application's lifecycle without needing direct kubectl access to every cluster."*

## 4. Operational Excellence (5 minutes)

### Application Health Dashboard
*Show healthy applications across both environments:*
- Resource utilization and pod status
- Route accessibility and health checks
- Policy compliance and security posture

### Environment Comparison
```
Environment 1 (Namespace-based):
├── frontend-team-dev: ✅ 1/1 pods (web-portal-dev.apps.cluster.local)
├── frontend-team-qa:  ✅ 2/2 pods (web-portal-qa.apps.cluster.local)
└── frontend-team-prod: ✅ 3/3 pods (web-portal.apps.cluster.local)

Environment 2 (Cluster-based):
├── Development Cluster: ✅ 1/1 pods (web-portal-dev.apps.dev-cluster.local)
├── QA Cluster:         ✅ 2/2 pods (web-portal-qa.apps.qa-cluster.local)
└── Production Cluster: ✅ 3/3 pods (web-portal.apps.prod-cluster.local)
```

### Production Auto-Scaling
*Demonstrate HPA in action and resource quota enforcement*

## 5. Summary & Value Proposition (2 minutes)

### Business Impact

#### For Application Teams:
- **Faster Time-to-Market**: Automated environment provisioning
- **Reduced Operational Overhead**: No infrastructure management required
- **Consistent Environments**: Dev/QA/Prod parity ensured
- **Self-Service Capabilities**: Deploy without platform team bottlenecks

#### For Platform Teams:
- **Centralized Governance**: Consistent policies across all environments
- **Reduced Manual Work**: Automated provisioning and compliance
- **Enhanced Security**: Enforced security standards and isolation
- **Unified Visibility**: Single dashboard for all applications and clusters

### Architectural Flexibility
*"We've demonstrated two common infrastructure patterns that organizations can choose based on their needs and scale. Organizations can start with Environment 1 and evolve to Environment 2 as they grow, using the same ACM ApplicationSets and governance policies."*

---

## Demo Access Information

### Environment 1 Applications:
- **Development**: https://web-portal-dev.apps.cluster.local
- **QA**: https://web-portal-qa.apps.cluster.local
- **Production**: https://web-portal.apps.cluster.local

### Environment 2 Applications (Cluster-Based):
- **Development**: https://web-portal-dev.apps.dev-cluster.local
- **QA**: https://web-portal-qa.apps.qa-cluster.local
- **Production**: https://web-portal.apps.prod-cluster.local

*Note: Domains are configurable through cluster labels and can be customized per deployment*

### Management Dashboards:
- **ACM Console**: https://multicloud-console.apps.hub-cluster.local
- **ArgoCD**: https://openshift-gitops-server-openshift-gitops.apps.cluster.local

---

## Additional Resources

- **Deployment Guide**: See `docs/deployment-guide.md` for detailed setup instructions
- **Repository Structure**: Explore the organized folder structure for both environments
- **Application Code**: Complete Kustomize configurations with environment overlays

*"Thank you for your attention. This demonstration shows how Red Hat ACM transforms the platform engineering experience, enabling both agility and governance at enterprise scale."*