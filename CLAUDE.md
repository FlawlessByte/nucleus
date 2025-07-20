# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Nucleus is a universal Helm chart designed to standardize Kubernetes deployments. It serves as a foundational chart that can be used as a dependency to deploy various types of applications with consistent patterns.

## Architecture

### Chart Structure
- **Base Chart**: Nucleus acts as a dependency/library chart that other charts inherit from
- **Multi-Component Support**: Single parent chart can define multiple microservices using different aliases
- **Template Abstraction**: Provides standardized templates for Deployments, StatefulSets, Services, HPA, KEDA, External Secrets, and more
- **Global Configuration**: Supports shared configuration across all components via `global.*` values

### Key Templates
- `deployment.yaml` - Supports both Deployment and StatefulSet workloads
- `service.yaml` - Service creation with multi-port support
- `external-secrets.yaml` - AWS Secrets Manager integration via External Secrets Operator
- `hpa.yaml` / `scaled-object.yaml` - Auto-scaling via HPA or KEDA
- `service-monitor.yaml` - Prometheus ServiceMonitor for observability
- `configmaps.yaml` - ConfigMap creation with support for multiple maps

### Global Features
- `global.env` - Shared environment variables across all components
- `global.image` - Common image configuration (repository, tag, pullPolicy)
- `global.additionalLabels` - Labels applied to all resources
- `global.nodeSelector`, `global.tolerations` - Scheduling constraints

## Development Commands

### Testing and Validation
```bash
# Lint the chart
helm lint .

# Test template rendering (requires valid values)
helm template test . --values path/to/test-values.yaml

# Debug template rendering issues
helm template test . --values path/to/test-values.yaml --debug

# Validate against Kubernetes API
helm template test . --values path/to/test-values.yaml | kubectl apply --dry-run=client -f -
```

### Dependency Management
```bash
# Update chart dependencies when used as dependency
helm dependency update

# Package the chart
helm package .
```

## Usage Patterns

### As Dependency Chart
Most commonly used pattern where nucleus is added as a dependency:

```yaml
# Chart.yaml
dependencies:
  - name: nucleus
    version: "^0.1.0"
    repository: "file://../../../../../ops/infra/helm/nucleus"
    alias: my-component
    condition: my-component.enabled
```

### Multi-Component Architecture
Reference implementation: `src/services/device-consumer/ops/helm/device-consumer/Chart.yaml`
- Uses 7 nucleus dependencies with different aliases
- Each alias represents a separate microservice component
- Shared configuration via global values, component-specific overrides in individual sections

### Values Structure
```yaml
global:
  image:
    repository: registry.flawlessbyte.dev/app
    tag: v1.0.0
  env:
    ENVIRONMENT: production
    DATABASE_URL: postgres://...

component-name:
  enabled: true
  workload:
    replicaCount: 3
  service:
    enabled: true
    ports:
      - name: http
        port: 8080
```

## Migration Guide

When migrating existing charts to nucleus:
1. Add nucleus as dependency in Chart.yaml
2. Move deployment templates to values configuration
3. Consolidate common values into global section
4. Test template generation thoroughly
5. Validate environment-specific value files
6. Clean up any template files that are no longer required after the migration

## Common Issues

### Template Parsing Errors
- Check YAML indentation in values files
- Ensure all required values are provided
- Use `--debug` flag to see rendered templates

### Environment Variable Conflicts
- Use `global.env` for shared variables
- Component-specific env takes precedence over global
- Avoid duplicate keys between global and local env

## File Reference

- `README.md` - Comprehensive usage documentation and examples
- `EXAMPLES.md` - Multi-component usage patterns and dependency examples
- `CHANGELOG.md` - Version history and breaking changes
- `values.yaml` - Default values and configuration schema