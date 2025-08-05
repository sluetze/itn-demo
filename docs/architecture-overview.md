# Architecture Overview

## Repository Structure

```
itn-demo/
├── README.md                                    # Repository overview
├── demo.md                                      # Enhanced demo script
├── environment-1-namespace-based/               # Single cluster, namespace separation
│   ├── platform-team/
│   │   ├── applicationset-namespace-setup.yaml # Namespace provisioning
│   │   ├── applicationset-web-portal.yaml      # Application deployment
│   │   └── governance-policies.yaml            # Security and compliance policies
│   └── application-team/                       # Kustomize base for team resources
│       ├── kustomization.yaml                  # Base Kustomize configuration
│       ├── namespace.yaml
│       ├── resourcequota.yaml
│       ├── rbac.yaml
│       ├── limitrange.yaml
│       └── overlays/                           # Environment-specific overlays
│           ├── dev/
│           │   ├── kustomization.yaml
│           │   ├── namespace-patch.yaml
│           │   └── resourcequota-patch.yaml
│           ├── qa/
│           │   ├── kustomization.yaml
│           │   ├── namespace-patch.yaml
│           │   └── resourcequota-patch.yaml
│           └── prod/
│               ├── kustomization.yaml
│               ├── namespace-patch.yaml
│               └── resourcequota-patch.yaml
├── environment-2-cluster-based/                # Multi-cluster setup
│   ├── platform-team/
│   │   ├── applicationset-cluster-setup.yaml   # Multi-cluster provisioning
│   │   ├── applicationset-web-portal.yaml      # Multi-cluster app deployment
│   │   ├── cluster-labels.yaml                 # Cluster labeling configuration
│   │   └── governance-policies.yaml            # Multi-cluster policies
│   └── application-team/                       # Kustomize base for cluster setup
│       ├── kustomization.yaml                  # Base Kustomize configuration
│       ├── namespace.yaml
│       ├── resourcequota.yaml
│       ├── rbac.yaml
│       ├── limitrange.yaml
│       └── overlays/                           # Cluster-specific overlays
│           ├── dev/
│           │   ├── kustomization.yaml
│           │   ├── namespace-patch.yaml
│           │   └── resourcequota-patch.yaml
│           ├── qa/
│           │   ├── kustomization.yaml
│           │   ├── namespace-patch.yaml
│           │   └── resourcequota-patch.yaml
│           └── prod/
│               ├── kustomization.yaml
│               ├── namespace-patch.yaml
│               └── resourcequota-patch.yaml
├── application/                                 # Web Portal application
│   ├── base/                                   # Base Kubernetes manifests
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── route.yaml
│   │   ├── configmap.yaml
│   │   └── kustomization.yaml
│   └── overlays/                               # Environment-specific configs
│       ├── dev/                               # Namespace-based dev
│       │   ├── kustomization.yaml
│       │   ├── deployment-patch.yaml
│       │   └── route-patch.yaml
│       ├── qa/                                # Namespace-based qa
│       │   ├── kustomization.yaml
│       │   ├── deployment-patch.yaml
│       │   └── route-patch.yaml
│       ├── prod/                              # Namespace-based prod
│       │   ├── kustomization.yaml
│       │   ├── deployment-patch.yaml
│       │   ├── route-patch.yaml
│       │   └── hpa-patch.yaml
│       ├── cluster-dev/                       # Cluster-based dev
│       │   ├── kustomization.yaml
│       │   ├── deployment-patch.yaml
│       │   └── route-patch.yaml
│       ├── cluster-qa/                        # Cluster-based qa
│       │   ├── kustomization.yaml
│       │   ├── deployment-patch.yaml
│       │   └── route-patch.yaml
│       └── cluster-prod/                      # Cluster-based prod
│           ├── kustomization.yaml
│           ├── deployment-patch.yaml
│           ├── route-patch.yaml
│           └── hpa-patch.yaml
└── docs/
    ├── deployment-guide.md                      # Step-by-step deployment
    └── architecture-overview.md                 # This file
```

## Environment Comparison

| Aspect | Environment 1 (Namespace-based) | Environment 2 (Cluster-based) |
|--------|----------------------------------|--------------------------------|
| **Infrastructure** | Single OpenShift cluster | Multiple OpenShift clusters |
| **Environment Separation** | Namespace postfix (dev, qa, prod) | Dedicated clusters per environment |
| **Resource Isolation** | Namespace-level quotas and limits | Cluster-level isolation |
| **ApplicationSet Generator** | List generator with static environments | Cluster generator with dynamic discovery |
| **Governance** | Namespace-scoped policies | Cluster-scoped policies |
| **Scaling** | Shared cluster resources | Dedicated cluster resources |
| **Complexity** | Lower operational overhead | Higher isolation, more complex |
| **Cost Model** | Resource sharing within cluster | Per-cluster cost allocation |
| **Use Case** | Small to medium organizations | Enterprise with strict isolation needs |

## Application Architecture

### Web Portal Application
- **Base Technology**: NGINX web server
- **Container Image**: `quay.io/redhattraining/hello-world-nginx`
- **Storage**: Stateless (no persistent volumes)
- **Access**: Web-accessible via OpenShift Routes
- **Security**: Non-root containers, security contexts
- **Health Checks**: Liveness, readiness, and startup probes

### Environment-Specific Configurations

#### Development Environment
- **Replicas**: 1
- **Resources**: 200m CPU, 256Mi memory (limits)
- **Features**: Debug mode enabled, verbose logging
- **TLS**: Edge termination, allows insecure traffic
- **URL**: `web-portal-dev.apps.cluster.local`

#### QA Environment
- **Replicas**: 2
- **Resources**: 500m CPU, 512Mi memory (limits)
- **Features**: Standard logging, testing mode
- **TLS**: Edge termination, redirects insecure traffic
- **URL**: `web-portal-qa.apps.cluster.local`

#### Production Environment
- **Replicas**: 3 (with HPA: 3-10)
- **Resources**: 1000m CPU, 1Gi memory (limits)
- **Features**: Optimized for performance, minimal logging
- **TLS**: Edge termination, strict security
- **URL**: `web-portal.apps.cluster.local`
- **Scaling**: Horizontal Pod Autoscaler based on CPU/memory

## Security Model

### Pod Security Standards
- **Enforcement Level**: Restricted
- **Security Context**: Non-root user (1001)
- **Capabilities**: All dropped
- **Filesystem**: Read-only root filesystem where possible

### Network Security
- **Network Policies**: Namespace isolation
- **Ingress**: Only from OpenShift ingress controllers
- **Egress**: Controlled outbound traffic
- **TLS**: Edge termination on all routes

### RBAC Model
- **Team-specific Roles**: Full access within assigned namespaces
- **Cluster-level Access**: Limited to necessary operations
- **Service Accounts**: Dedicated accounts for automation
- **Cross-cluster**: Image pulling capabilities

## GitOps Workflow

### Repository Structure
1. **Base Manifests**: Common application components
2. **Environment Overlays**: Kustomize patches for each environment
3. **ApplicationSets**: Automated deployment across environments
4. **Governance Policies**: Security and compliance enforcement

### Deployment Flow
1. **Code Change**: Developer commits application changes
2. **GitOps Sync**: ArgoCD detects repository changes
3. **Multi-Environment Deploy**: ApplicationSet generates applications
4. **Policy Validation**: ACM enforces governance policies
5. **Health Monitoring**: Continuous application health checks

## Governance and Compliance

### Policy Framework
- **Security Policies**: Container security, pod security standards
- **Resource Policies**: CPU/memory limits, storage quotas
- **Network Policies**: Traffic isolation and segmentation
- **Compliance Reporting**: Automated violation detection and remediation

### Multi-Cluster Governance
- **Consistent Policies**: Same security standards across all clusters
- **Environment-Specific Rules**: Production gets additional restrictions
- **Automated Remediation**: Self-healing policy enforcement
- **Compliance Dashboard**: Centralized view of policy compliance

## Operational Considerations

### Monitoring and Observability
- **Application Health**: Pod status, resource utilization
- **Policy Compliance**: Real-time compliance monitoring
- **GitOps Status**: Sync status across all applications
- **Multi-Cluster View**: Unified dashboard for all environments

### Scaling Strategies
- **Environment 1**: Vertical scaling within shared cluster
- **Environment 2**: Horizontal scaling with dedicated clusters
- **Auto-scaling**: HPA for production workloads
- **Resource Management**: Quotas prevent resource contention

### Disaster Recovery
- **GitOps Recovery**: All configurations stored in Git
- **Multi-Cluster Redundancy**: Environment 2 provides natural DR
- **Backup Strategy**: Namespace-level or cluster-level backups
- **Recovery Procedures**: Automated restoration via ApplicationSets

## Next Steps and Evolution

### Migration Path
1. **Start Small**: Begin with Environment 1 (namespace-based)
2. **Prove Concept**: Validate GitOps and governance processes
3. **Scale Up**: Add more teams and applications
4. **Evolve Architecture**: Migrate to Environment 2 as needed
5. **Advanced Features**: Add monitoring, service mesh, CI/CD integration

### Advanced Configurations
- **Multi-Region Clusters**: Geographic distribution
- **Hybrid Cloud**: Mix of on-premise and cloud clusters
- **Edge Computing**: IoT and edge deployment scenarios
- **Service Mesh**: Istio integration for advanced traffic management
- **Observability Stack**: Prometheus, Grafana, Jaeger integration

## Configuration Management with Kustomize

### Overview
The demo uses Kustomize for all configuration management instead of Helm, providing:

- **Environment Variables**: Configurable through ConfigMaps
- **Namespace Names**: Customizable per environment
- **Route Hostnames**: Environment and cluster-specific domains
- **Resource Quotas**: Environment-specific limits and requests
- **Cluster Labels**: Multi-cluster configuration through ACM labels

### Configurable Variables

#### Environment 1 (Namespace-based):
```yaml
configMapGenerator:
- name: environment-config
  literals:
  - NAMESPACE_NAME=frontend-team-{dev,qa,prod}
  - ROUTE_HOST=web-portal-{env}.apps.cluster.local
  - CLUSTER_DOMAIN=apps.cluster.local
  - ENVIRONMENT={dev,qa,prod}
  - CPU_LIMIT={2,4,8}
  - MEMORY_LIMIT={4Gi,8Gi,16Gi}
```

#### Environment 2 (Cluster-based):
```yaml
configMapGenerator:
- name: cluster-config
  literals:
  - CLUSTER_NAME={development,qa,production}-cluster
  - ROUTE_HOST=web-portal-{env}.apps.{env}-cluster.local
  - CLUSTER_DOMAIN=apps.{env}-cluster.local
  - ENVIRONMENT={dev,qa,prod}
```

### Kustomize Replacements
Dynamic configuration using Kustomize replacements:

```yaml
replacements:
- source:
    kind: ConfigMap
    name: environment-config
    fieldPath: data.ROUTE_HOST
  targets:
  - select:
      kind: Route
      name: web-portal
    fieldPaths:
    - spec.host
```

### Customization Guide

#### Changing Cluster Domains
1. Update cluster labels in `cluster-labels.yaml`:
   ```yaml
   labels:
     domain: apps.custom-domain.com
   ```
2. Modify overlay ConfigMaps to reflect new domain
3. ApplicationSets automatically pick up new domain values

#### Adding New Environments
1. Create new overlays: `application/overlays/{new-env}/`
2. Configure environment-specific variables
3. Update ApplicationSet generators
4. Deploy with `oc apply`

#### Modifying Resource Quotas
1. Edit overlay patches in `overlays/{env}/resourcequota-patch.yaml`
2. Update ConfigMap generators with new limits
3. Changes propagate automatically via GitOps