# Changelog

All notable changes to the Nucleus Helm chart will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2025-01-XX

### Added
- Support for Kubernetes Secrets via `additionalSecrets` configuration
- Support for Kubernetes CronJobs via `additionalCronJobs` configuration
- Support for generic additional Kubernetes resources via `additionalResources` configuration
- `commonLabels` configuration to apply labels across all resources (similar to `commonAnnotations`)
- `containerName` parameter to customize container name (defaults to Release name)
- Multi-line annotation support using YAML literal block scalar syntax (`|`)
- Enhanced documentation with examples for Secrets, CronJobs, and additional resources

### Changed
- Container name now defaults to Release name instead of Chart name
- Annotations now support multi-line values using YAML literal block syntax
- Improved label merging to support both `global.additionalLabels` and `commonLabels`

### Fixed
- Removed root-level `replicaCount` parameter (now only uses `workload.replicaCount`)
- Fixed container name inheritance from `nameOverride` and `fullnameOverride`

## [1.0.0] - 2025-07-20

### Added
- Initial release of Nucleus Helm chart
- Flexible workload support (Deployment/StatefulSet) 
- Auto-scaling with HPA and KEDA support
- External Secrets integration with AWS Secrets Manager
- Service management with multi-port support
- Security context and RBAC configuration
- Health check probes and pod disruption budgets
- Init containers and sidecar support
- ConfigMap management
- ServiceMonitor support for Prometheus