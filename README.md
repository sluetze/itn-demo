# Red Hat ACM Demo - Self-Service Platform

This repository contains the complete setup for demonstrating Red Hat Advanced Cluster Management (ACM) with two different infrastructure approaches.

## Repository Structure

```
├── demo.md                              # Enhanced demo script
├── environment-1-namespace-based/       # Namespace-based environment separation
├── environment-2-cluster-based/         # Cluster-based environment separation
├── application/                         # Web Portal application manifests
└── docs/                               # Additional documentation
```

## Quick Start

1. **Environment 1**: Namespace-based separation (single cluster)
2. **Environment 2**: Cluster-based separation (multiple clusters with ACM labels)

Each environment includes:
- Platform team setup (ApplicationSets, governance policies)
- Application team resources (namespaces, RBAC, quotas)
- Sample web application (Web Portal)

## Prerequisites

- OpenShift clusters with Red Hat ACM installed
- Git repository access
- Cluster admin permissions for initial setup

## Demo Flow

Follow the enhanced `demo.md` script for a complete walkthrough of both environments.