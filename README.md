# k8s-test

Example repository demonstrating the integration of [k8s-bootstrap-buildkite-plugin](https://github.com/Mykematt/k8s-bootstrap-buildkite-plugin) with Buildkite Secrets.

## Overview

This pipeline demonstrates:

1. **Securely load kubeconfig** using native Buildkite Secrets
2. **Create scoped service accounts** using k8s-bootstrap plugin  
3. **Deploy applications** using the generated scoped kubeconfig

## Integration Overview

This pipeline demonstrates the secure integration of the k8s-bootstrap plugin with Buildkite Secrets using a three-step architecture:

**Step 1: Load Secrets** (`queue: "default"`)

- Uses native `buildkite-agent secret get` command to load kubeconfig from Buildkite Secrets
- Runs on regular agents with internet access to Buildkite API
- Environment variable persists to downstream steps

**Step 2: Bootstrap Service Account** (`queue: "kubernetes"`)

- Uses the `k8s-bootstrap` plugin with the loaded secret from Step 1
- Runs on K8s agents with direct cluster access for kubectl operations
- Creates scoped service account and uploads kubeconfig artifact

**Step 3: Deploy Application** (`queue: "kubernetes"`)

- Downloads kubeconfig artifact from Step 2
- Runs kubectl apply commands with scoped permissions
- Runs on K8s agents for optimal deployment performance

## Setup Instructions

1. **Create Buildkite Secret:**
   - Go to your Buildkite Organization Settings → Secrets
   - Create a new secret with key: `k8s_admin_kubeconfig`
   - Paste your cluster admin kubeconfig YAML as the value

2. **Configure Agent Queues:**
   - Ensure you have agents in `queue=default` (regular agents with internet access)
   - Ensure you have agents in `queue=kubernetes` (K8s agents with cluster access)

3. **Plugin Integration:**
   ```yaml
   # Step 1: Load secret natively
   command: |
     export KUBECONFIG_SECRET="$(buildkite-agent secret get k8s_admin_kubeconfig)"
   
   # Step 2: Use in k8s-bootstrap plugin
   plugins:
     - Mykematt/k8s-bootstrap#v1.0.0:
         cluster-admin-kubeconfig: "${KUBECONFIG_SECRET}"
   ```

## Security Benefits

✅ **Principle of Least Privilege**: Deployments use scoped service account, not cluster-admin  
✅ **Secret Management**: Admin kubeconfig stored securely in Buildkite Secrets  
✅ **Temporary Credentials**: Service account tokens can be rotated automatically  
✅ **Audit Trail**: All operations logged through Buildkite pipeline  

## Files

- `.buildkite/pipeline.yml` - Main pipeline configuration
- `k8s/` - Kubernetes manifests for nginx deployment
  - `configmap.yaml` - Configuration data
  - `deployment.yaml` - Nginx deployment
  - `service.yaml` - Service to expose nginx

## Usage

1. Ensure you have the admin kubeconfig secret configured in Buildkite
2. Run the pipeline in Buildkite
3. The first step creates a scoped service account
4. The second step deploys using only the scoped permissions

This demonstrates a secure, production-ready pattern for Kubernetes deployments in CI/CD pipelines.
