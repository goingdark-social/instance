# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Kubernetes infrastructure repository for running goingdark.social, containing manifests for deploying Mastodon with supporting services. The repository uses Kustomize for configuration management and is structured for ArgoCD deployment.

## Architecture

The repository follows a GitOps pattern with two main applications:

### Mastodon
Complete social media platform deployment including:
- **Web servers** (`mastodon/web/`) - Main Mastodon application (2 replicas, glitch-soc/mastodon:v4.4.3)
- **Background jobs** (`mastodon/sidekiq/`) - Handles async processing
- **Streaming API** (`mastodon/streaming/`) - Real-time updates
- **PostgreSQL cluster** (`mastodon/postgres/`) - Primary database with connection pooling and backup (Zalando Postgres Operator)
- **Redis** (`mastodon/redis/`) - Caching and session storage
- **Elasticsearch** (`mastodon/elasticsearch/`) - Full-text search with Kibana
- **Nginx proxy** (`mastodon/proxy/`) - Reverse proxy and static content

Custom configuration:
- Posts limited to 1000 characters (vs standard 500)
- Reply fetching: 5min initial delay, 15min intervals, caps at 1000 total replies

### Hypebot
Automated boost bot (`hypebot/`) running `ghcr.io/goingdark-social/hypebot:v0.1.0` that automatically boosts popular posts.

## Development Commands

### Validation and Dry-run
```bash
# Validate all manifests
kustomize build . --dry-run=server

# Validate specific component
kustomize build mastodon --dry-run=server
kustomize build hypebot --dry-run=server
```

### Deployment
```bash
# Deploy everything
kubectl apply -k .

# Deploy specific components
kubectl apply -k mastodon
kubectl apply -k hypebot

# Check deployment status
kubectl get pods -n mastodon
kubectl get pods -n hypebot
```

### Monitoring and Debugging
```bash
# Check application status
kubectl get applications -n argocd

# View logs
kubectl logs -n mastodon -l app=mastodon-web
kubectl logs -n mastodon -l app=mastodon-sidekiq
kubectl logs -n hypebot -l app=hypebot

# Check database status
kubectl get postgresql -n mastodon
kubectl describe postgresql mastodon-postgresql -n mastodon
```

## Key Configuration Files

- `project.yaml` - ArgoCD AppProject definition
- `kustomization.yaml` - Root Kustomize configuration
- `mastodon/kustomization.yaml` - Mastodon component orchestration
- `mastodon/postgres/database.yaml` - PostgreSQL cluster with backup configuration
- `mastodon/web/web-deployment.yaml` - Main application deployment

## Security Configuration

The deployment includes comprehensive security measures:
- TLS certificates managed via cert-manager
- External secrets integration for credential management
- Network policies for service isolation
- Security contexts with non-root users and capability dropping
- Resource limits and tolerations for autoscaler nodes

## Important Notes

- All container images are pinned to specific versions for stability
- Database uses SSL with certificate verification (verify-ca mode)
- WAL-G backup system configured for PostgreSQL disaster recovery
- Elasticsearch uses custom CA certificates
- VPA (Vertical Pod Autoscaler) setup available but not automatically applied