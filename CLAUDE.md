# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Helm chart repository for managing ArgoCD ApplicationSets and Applications that deploy Kubernetes infrastructure components and business applications across multiple environments. The repository follows a GitOps pattern using ArgoCD for continuous deployment.

## Key Commands

### Helm Operations
```bash
# Validate Helm chart syntax
helm lint .

# Template and validate chart output
helm template . --values values/dev/lgm.yaml --values values/dev/common.yaml

# Dry run deployment
helm install --dry-run --debug argocd-chart . --values values/dev/lgm.yaml

# Package chart
helm package .
```

### Git Operations
```bash
# Check repository status
git status

# Validate YAML files
find . -name "*.yaml" -o -name "*.yml" | xargs -I {} yamllint {}
```

## Architecture

### Repository Structure
- **Root Level**: Main Helm chart (`Chart.yaml`) that orchestrates all ApplicationSets
- **templates/v1/**: Contains ArgoCD ApplicationSet and Application definitions
- **values/**: Environment-specific configuration files organized by stage (dev, staging, prod)
- **k8s-component-addons/**: Infrastructure components (cert-manager, envoy, grafana, external-secrets)
- **k8s-components/**: Platform components with chart definitions
- **lgm/**: LGM-specific application charts
- **biz-apps/**: Business application deployments
- **customers/**: Customer-specific configurations
- **argocd-repo-creds/**: ArgoCD repository credentials setup
- **vdi-auth-app/**: VDI authentication application

### Component Types

1. **Infrastructure Components** (`k8s-component-addons/`):
   - cert-manager: TLS certificate management
   - envoy: Load balancing and ingress
   - grafana: Monitoring and observability
   - external-secrets: Secret management

2. **Platform Components** (`k8s-components/` and `lgm/`):
   - traefik: Reverse proxy and load balancer
   - external-secrets: Kubernetes secret management
   - nats/nack: Messaging infrastructure
   - fluent-bit: Log collection and forwarding
   - loki/mimir: Observability stack

### Environment Management

The repository supports multi-environment deployments:
- **dev/**: Development environment configurations
- **staging/**: Staging environment configurations
- **prod/**: Production environment configurations

Each environment has:
- `lgm.yaml`: LGM-specific settings including repo URLs, load balancer configs, SNS topics
- `common.yaml`: Shared configuration across environments
- `parent.yaml`: Parent environment references

### ArgoCD Integration

The main ApplicationSet template (`templates/v1/appsets.yaml`) creates:
1. **LGM Components ApplicationSet**: Deploys charts defined in `lgm/all-charts.yaml`
2. **K8s Component Addons Application**: Deploys infrastructure components

Key features:
- Environment-aware naming (prefixed with environment name)
- Automated sync with prune and self-heal enabled
- Multi-source support for chart repositories and values
- Server-side apply for better resource management

## Configuration Patterns

### Values File Hierarchy
```
values/values.yaml (global defaults)
├── values/{stage}/common.yaml (stage-specific common config)
├── values/{stage}/lgm.yaml (LGM-specific config)
└── k8s-component-addons/values/values.yaml (component defaults)
```

### Chart Definition Format
Charts are defined in YAML files with this structure:
```yaml
- name: component-name
  repoUrl: https://helm-repo-url
  chartName: chart-name
  targetRevision: "version"
  namespace: target-namespace
  valuesFile: values-file.yaml
```

### Environment Variables
- `environment`: Environment prefix for resource naming
- `stage`: Environment stage (dev/staging/prod)
- `repoUrl.templates`: Git repository URL for templates
- `repoBranch.templates`: Git branch for templates

## Development Workflow

1. **Add New Component**:
   - Add chart definition to appropriate `all-charts.yaml`
   - Create values file in `values/{environment}/`
   - Update component configuration in `k8s-component-addons/values/values.yaml` if needed

2. **Modify Environment**:
   - Update values in `values/{stage}/lgm.yaml` or `values/{stage}/common.yaml`
   - Test with `helm template` before committing

3. **Infrastructure Changes**:
   - Modify templates in `k8s-component-addons/templates/v1/`
   - Update corresponding values files

4. **Validation**:
   - Always validate YAML syntax
   - Test Helm templating with target environment values
   - Verify ArgoCD ApplicationSet generation