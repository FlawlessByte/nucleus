# Changelog

All notable changes to the Nucleus Helm chart will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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