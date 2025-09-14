# goingdark.social Kubernetes Infrastructure

This is a Kubernetes infrastructure repository for running goingdark.social, containing manifests for deploying Mastodon with supporting services (Hypebot and Cryptpad). The repository uses Kustomize for configuration management and is structured for ArgoCD GitOps deployment.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Tool Validation
- Ensure you have the required tools:
  - `kubectl version --client` - Kubernetes CLI (v1.34.0)
  - `kustomize version` - Configuration management (v5.7.1)
  - `yamllint --version` - YAML linting
  - `yq --version` - YAML processing
  - `docker --version` - Container runtime

### Repository Validation and Building
- Validate all manifests: `kustomize build .` -- takes <1 second. NEVER CANCEL.
- Validate specific components:
  - `kustomize build mastodon` -- takes <1 second. NEVER CANCEL.
  - `kustomize build hypebot` -- takes <1 second. NEVER CANCEL.
  - `kustomize build cryptpad` -- takes <1 second. NEVER CANCEL.
- YAML linting: `yamllint .` -- takes <1 second. NEVER CANCEL.
  - Note: This will show style issues (missing document starts, line length) which is normal for Kubernetes manifests
  - Use `yamllint . --no-warnings` to see only errors

### Manual Validation Requirements
Since this is a Kubernetes infrastructure repository without a running cluster:
- ALWAYS run `kustomize build .` after making changes to validate YAML syntax
- ALWAYS run `yamllint .` to check for common YAML formatting issues
- Use `yq` to validate and query specific YAML structures when debugging
- Validate image tags and references exist before committing changes

## Repository Structure

### Root Components (90 YAML files total)
- `project.yaml` - ArgoCD AppProject definition for GitOps
- `kustomization.yaml` - Root Kustomize configuration orchestrating all apps
- `mastodon/` - Complete Mastodon social media platform (main application)
- `hypebot/` - Automated boost bot for trending posts
- `cryptpad/` - Collaborative document editing platform

### Mastodon Components
Complex deployment with 12 sub-components:
- `mastodon/web/` - Main Mastodon web application (2 replicas, glitch-soc/mastodon:v4.4.3)
- `mastodon/sidekiq/` - Background job processing
- `mastodon/streaming/` - Real-time API for live updates
- `mastodon/postgres/` - PostgreSQL cluster with Zalando operator, backup via WAL-G
- `mastodon/redis/` - Caching and session storage (16 databases)
- `mastodon/elasticsearch/` - Full-text search with Kibana dashboard
- `mastodon/proxy/` - Nginx reverse proxy for static content
- `mastodon/base/` - Core namespaces, secrets, and service accounts
- `mastodon/networking/` - HTTPRoutes and ingress configuration
- `mastodon/observability/` - Prometheus monitoring and metrics
- `mastodon/cluster/` - Priority classes and cluster-wide resources

## Validation Scenarios

### After Making Changes
1. **Syntax Validation**: Run `kustomize build .` to ensure YAML is valid
2. **Component Validation**: Run `kustomize build [component]` for specific components
3. **Style Validation**: Run `yamllint . --no-warnings` to check critical issues
4. **Image Validation**: Verify container image tags exist and are accessible
5. **Reference Validation**: Check that secret/configmap references are correct

### Common Change Patterns
- **Version Updates**: Change container image tags in deployment files
- **Configuration Changes**: Modify ConfigMaps for application settings
- **Resource Scaling**: Adjust replica counts, CPU/memory limits
- **Secret Management**: Update ExternalSecret references for credentials
- **Networking Changes**: Modify HTTPRoute configurations for ingress

## Common Tasks

### Exploring the Repository
- View root directory: `ls -la` shows project.yaml, kustomization.yaml, and component dirs
- Count YAML files: `find . -name "*.yaml" -o -name "*.yml" | wc -l` (90 files)
- Find deployment files: `find . -name "*deploy*.yaml" -o -name "*deployment*.yaml"`
- Search for specific configs: `grep -r "image:" . | grep -v ".git"`

### Component-Specific Tasks
- **Hypebot Updates**: Edit `hypebot/deploy.yaml` for image tags or `hypebot/app-config.yaml` for bot configuration
- **Mastodon Scaling**: Modify replica counts in `mastodon/web/web-deployment.yaml` or `mastodon/sidekiq/sidekiq-deployment.yaml`
- **Database Changes**: Update `mastodon/postgres/database.yaml` for PostgreSQL cluster configuration
- **Monitoring**: Check `mastodon/observability/` for Prometheus ServiceMonitor configurations

### Key Configuration Files Referenced in CLAUDE.md
- `project.yaml` - ArgoCD AppProject definition
- `kustomization.yaml` - Root Kustomize configuration
- `mastodon/kustomization.yaml` - Mastodon component orchestration
- `mastodon/postgres/database.yaml` - PostgreSQL cluster with backup configuration
- `mastodon/web/web-deployment.yaml` - Main application deployment

## Important Constraints and Notes

### This Repository Cannot/Does Not:
- **Build applications** - No build system, Makefile, or programming language dependencies
- **Run tests** - No test suite or CI/CD workflows
- **Deploy directly** - Requires a Kubernetes cluster and kubectl context
- **Validate with kubectl** - Server-side validation needs cluster access

### This Repository Can/Does:
- **Validate manifests** - Using kustomize build and yamllint
- **Generate manifests** - Output complete Kubernetes YAML for deployment
- **Lint YAML** - Check syntax and style issues
- **Process YAML** - Query and modify using yq

### Security and Production Notes
- All container images are pinned to specific versions for stability
- Database uses SSL with certificate verification (verify-ca mode)  
- WAL-G backup system configured for PostgreSQL disaster recovery
- Network policies ensure service isolation
- External secrets integration for credential management
- VPA (Vertical Pod Autoscaler) configurations available but not auto-applied

### Custom Mastodon Configuration
- Posts limited to 1000 characters (vs standard 500)
- Reply fetching: 5min initial delay, 15min intervals, caps at 1000 total replies
- Elasticsearch uses custom CA certificates
- Redis configured with 16 databases and security restrictions

## Troubleshooting

### Common Issues
- **YAML Syntax Errors**: Use `kustomize build .` to identify malformed YAML
- **Reference Errors**: Check that ConfigMap/Secret names match between files
- **Image Tag Issues**: Verify container images exist in registries
- **Kustomize Errors**: Ensure all resource references in kustomization.yaml files exist

### Debugging Commands
- **View generated manifests**: `kustomize build . | less` (pipe through less for large output)
- **Check specific resources**: `kustomize build . | yq 'select(.kind == "Deployment")'`
- **Find resource by name**: `kustomize build . | yq 'select(.metadata.name == "mastodon-web")'`
- **Validate specific namespaces**: `kustomize build . | yq 'select(.metadata.namespace == "mastodon")'`

Remember: This is GitOps infrastructure - changes here affect production deployments via ArgoCD.