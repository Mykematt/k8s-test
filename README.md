# k8s-test

Example repository demonstrating the integration of [k8s-bootstrap-buildkite-plugin](https://github.com/Mykematt/k8s-bootstrap-buildkite-plugin) with [cluster-secrets-buildkite-plugin](https://github.com/buildkite-plugins/cluster-secrets-buildkite-plugin).

## Overview

This project shows how to:
1. **Securely load kubeconfig** using cluster-secrets plugin
2. **Create scoped service accounts** using k8s-bootstrap plugin  
3. **Deploy applications** using the generated scoped kubeconfig

## Pipeline Flow

### Step 1: Create Service Account
- Uses `cluster-secrets#v1.0.0` to load admin kubeconfig from Buildkite Secrets
- Uses `k8s-bootstrap#v1.0.0` to create a service account with limited permissions
- Uploads scoped kubeconfig as artifact

### Step 2: Deploy Application
- Downloads the scoped kubeconfig artifact
- Deploys nginx application using the **scoped** kubeconfig (not admin)

## Setup Requirements

### 1. Buildkite Secret
Create a secret in your Buildkite organization:
- **Key**: `k8s-admin-kubeconfig`
- **Value**: Raw kubeconfig YAML with cluster-admin permissions

### 2. Plugin Dependencies
This pipeline uses:
- `cluster-secrets#v1.0.0` - Load secrets from Buildkite
- `Mykematt/k8s-bootstrap#v1.0.0` - Create scoped service accounts

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
