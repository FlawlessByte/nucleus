# Using Nucleus Chart as a Dependency

This guide explains how to use the Nucleus chart as a dependency in your Helm charts.

## Adding Nucleus as a Dependency

### 1. Update Chart.yaml

```yaml
# Chart.yaml
apiVersion: v2
name: my-application
version: 0.1.0

dependencies:
  - name: nucleus
    version: "^1.0.0"
    repository: "https://flawlessbyte.github.io/nucleus"
    alias: app
    condition: app.enabled
```

### 2. Update Dependencies

```bash
helm dependency update
```

## Basic Usage

### Single Component

#### values.yaml
```yaml
app:
  enabled: true
  image:
    repository: nginx
    tag: "1.21"
  
  containerPorts:
    - name: http
      containerPort: 80

  service:
    enabled: true
    ports:
      - name: http
        port: 80
        targetPort: 80

  resources:
    requests:
      cpu: 100m
      memory: 128Mi
```

## Multi-Component Example

### Chart.yaml
```yaml
dependencies:
  - name: nucleus
    version: "^1.0.0"
    repository: "https://flawlessbyte.github.io/nucleus"
    alias: frontend
    condition: frontend.enabled
  - name: nucleus
    version: "^1.0.0"
    repository: "https://flawlessbyte.github.io/nucleus"
    alias: api
    condition: api.enabled
  - name: nucleus
    version: "^1.0.0"
    repository: "https://flawlessbyte.github.io/nucleus"
    alias: redis
    condition: redis.enabled
```

### values.yaml
```yaml
# Frontend
frontend:
  enabled: true
  image:
    repository: myapp/frontend
    tag: "v1.0.0"
  
  service:
    enabled: true
    type: LoadBalancer
    ports:
      - name: http
        port: 80
        targetPort: 3000
  
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 10

# API Backend
api:
  enabled: true
  image:
    repository: myapp/api
    tag: "v1.0.0"
  
  service:
    enabled: true
    ports:
      - name: http
        port: 8080
        targetPort: 8080
  
  externalSecrets:
    enabled: true
    dataFrom:
      - extract:
          key: myapp/secrets

# Redis Cache
redis:
  enabled: true
  image:
    repository: redis
    tag: "7-alpine"
  
  workload:
    kind: StatefulSet
  
  service:
    enabled: true
    ports:
      - name: redis
        port: 6379
        targetPort: 6379
  
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 5Gi
```

## Environment-Specific Values

### values-dev.yaml
```yaml
frontend:
  workload:
    replicaCount: 1
  autoscaling:
    enabled: false

api:
  workload:
    replicaCount: 1
  externalSecrets:
    enabled: false

redis:
  enabled: false
```

### values-prod.yaml
```yaml
frontend:
  autoscaling:
    maxReplicas: 20
  podDisruptionBudget:
    enabled: true

api:
  resources:
    limits:
      cpu: 1000m
      memory: 1Gi
  podDisruptionBudget:
    enabled: true

redis:
  resources:
    limits:
      memory: 512Mi
```

## Shared Configuration

### Global Configuration with `global.additionalConfigMaps`

Share ConfigMaps across all charts using the nucleus base chart:

```yaml
# Parent chart values.yaml
global:
  image:
    repository: myapp/service
    tag: latest
    pullPolicy: IfNotPresent

  # Global ConfigMaps shared across all subcharts
  additionalConfigMaps:
  - name: shared-config
    data:
      environment: "production"
      database-url: "postgres://shared-db:5432/app"
      redis-url: "redis://shared-redis:6379"
      log-level: "info"
    labels:
      scope: global
      environment: production
    annotations:
      shared.flawlessbyte.dev/config: "true"
  
  - name: feature-flags
    data:
      feature-a: "enabled"
      feature-b: "disabled"
      rollout-percentage: "50"
    labels:
      type: feature-flags

  # Global ExternalSecrets shared across all subcharts
  externalSecrets:
  - name: shared-database-credentials
    secretStoreRef:
      name: aws-secrets-manager-store
      kind: ClusterSecretStore
    target:
      creationPolicy: Owner
    dataFrom:
      - extract:
          key: shared/database/production-credentials
          version: "uuid/12345678-1234-5678-9abc-def012345678"
    refreshInterval: "1h"
    labels:
      scope: global
      environment: production
    annotations:
      shared.flawlessbyte.dev/secret: "true"
  
  - name: shared-api-keys
    secretStoreRef:
      name: aws-secrets-manager-store
      kind: ClusterSecretStore
    target:
      creationPolicy: Owner
    dataFrom:
      - extract:
          key: shared/api-keys/third-party
          version: "uuid/87654321-4321-8765-dcba-210987654321"
    refreshInterval: "30m"
    labels:
      type: api-keys
      scope: global

# Individual components can still have their own ConfigMaps and ExternalSecrets
frontend:
  enabled: true
  additionalConfigMaps:
  - name: frontend-config
    data:
      api-endpoint: "/api/v1"
      theme: "dark"
  
  externalSecrets:
    enabled: true
    secretStoreRef:
      name: aws-secrets-manager-store
      kind: ClusterSecretStore
    target:
      creationPolicy: Owner
    dataFrom:
      - extract:
          key: frontend/oauth-secrets
          version: "uuid/frontend-secrets-123"

api:
  enabled: true
  additionalConfigMaps:
  - name: api-config
    data:
      max-connections: "100"
      timeout: "30s"

  externalSecrets:
    enabled: true
    secretStoreRef:
      name: aws-secrets-manager-store
      kind: ClusterSecretStore
    target:
      creationPolicy: Owner
    dataFrom:
      - extract:
          key: api/jwt-secrets
          version: "uuid/api-secrets-456"
```

### YAML Anchors for Common Settings

```yaml
common: &common
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
  podDisruptionBudget:
    enabled: true

frontend:
  <<: *common
  enabled: true
  image:
    repository: myapp/frontend

api:
  <<: *common
  enabled: true
  image:
    repository: myapp/api
  resources:
    requests:
      cpu: 200m
      memory: 256Mi
```

## Deployment Commands

```bash
# Install with development values
helm install myapp . -f values.yaml -f values-dev.yaml

# Install with production values
helm install myapp . -f values.yaml -f values-prod.yaml

# Upgrade specific component
helm upgrade myapp . --set api.image.tag=v1.1.0 --reuse-values

# Enable/disable components
helm upgrade myapp . --set redis.enabled=false --reuse-values
```

## Testing

```bash
# Render templates
helm template myapp . -f values.yaml

# Dry run
helm install myapp . -f values.yaml --dry-run

# Update dependencies
helm dependency update
```

## Best Practices

1. **Use meaningful aliases** that match your component names
2. **Always use conditions** to enable/disable components
3. **Keep component configurations separate** in your values
4. **Use environment-specific value files** for different deployments
5. **Test with `helm template`** before deploying

## Common Patterns

### Database with Init Container
```yaml
database:
  enabled: true
  image:
    repository: postgres
    tag: "13"
  
  initContainers:
    - name: init-db
      image: myapp/migrations
      command: ["migrate", "up"]
  
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 20Gi
```

### Worker with KEDA Scaling
```yaml
worker:
  enabled: true
  image:
    repository: myapp/worker
  
  service:
    enabled: false
  
  autoscaling:
    enabled: true
    type: keda
    keda:
      triggers:
        - type: redis
          metadata:
            address: myapp-redis:6379
            listName: jobs
            listLength: "5"
```

### Application with Secrets and CronJobs
```yaml
app:
  enabled: true
  image:
    repository: myapp/api
    tag: v1.0.0
  
  # Create application secrets
  additionalSecrets:
    - name: app-secrets
      type: Opaque
      stringData:
        database-url: postgres://db:5432/myapp
        api-key: my-secret-api-key
        jwt-secret: my-jwt-secret-key
  
  # Create scheduled backup job
  additionalCronJobs:
    - name: daily-backup
      schedule: "0 2 * * *"  # Daily at 2 AM
      concurrencyPolicy: Forbid
      jobTemplate:
        template:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: backup-tool:latest
            command: ["/bin/sh", "-c"]
            args: ["backup.sh"]
            env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: database-url
            volumeMounts:
            - name: backup-storage
              mountPath: /backups
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
  
  # Create additional resources
  additionalResources:
    - apiVersion: v1
      kind: PersistentVolumeClaim
      name: backup-pvc
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 20Gi
```