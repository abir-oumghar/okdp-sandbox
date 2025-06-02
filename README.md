# okdp-sandbox
# OKDP Sandbox Installation Guide

[![Kubernetes](https://img.shields.io/badge/kubernetes-1.28+-blue.svg)](https://kubernetes.io/)
[![Flux](https://img.shields.io/badge/flux-2.4.0+-purple.svg)](https://fluxcd.io/)
[![Kind](https://img.shields.io/badge/kind-latest-orange.svg)](https://kind.sigs.k8s.io/)

Welcome to the OKDP Sandbox Installation Guide. This guide provides instructions for setting up a local Kubernetes development environment with all the components needed for the OKDP sandbox.

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Cluster Setup](#-cluster-setup)
- [Flux Configuration](#-flux-configuration)
- [DNS Configuration](#-dns-configuration)
- [Certificate Setup](#-certificate-setup)
- [Accessing Components](#-accessing-components)
- [Authentication & Credentials](#-authentication--credentials)
- [Troubleshooting](#-troubleshooting)

## 📦 Prerequisites

Before you begin, ensure you have the following tools installed:

- [Flux](https://fluxcd.io/) version 2.4.0 or later
- [Kind](https://kind.sigs.k8s.io/) (Kubernetes in Docker)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- GitHub account with permissions to create repositories

## 🚀 Cluster Setup

Create a Kind cluster configuration file:

**For MacOS and Linux:**


```bash
cat >/tmp/kind-config.yaml <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kind-okdp
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30080
    hostPort: 80
    protocol: TCP
  - containerPort: 30443
    hostPort: 443
    protocol: TCP
EOF

kind create cluster --config /tmp/kind-config.yaml
```

**For Windows:**

```powershell
# Create config file
$kindConfig = @"
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kind-okdp
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30080
    hostPort: 80
    protocol: TCP
  - containerPort: 30443
    hostPort: 443
    protocol: TCP
"@
$kindConfig | Out-File -FilePath $env:TEMP\kind-config.yaml -Encoding ascii

# Create cluster
kind create cluster --config $env:TEMP\kind-config.yaml
```

This configuration creates a local Kubernetes cluster with a single control plane node and maps ports for HTTP and HTTPS.

## 🔄 Flux Configuration

### 1. Create a GitHub Token

Create a GitHub Personal Access Token with `repo` permissions. Visit GitHub's [token settings page](https://github.com/settings/tokens) to create one.

### 2. Export the Token

**For MacOS and Linux:**
```bash
export GITHUB_TOKEN=your_github_token
```

**For Windows:**
```powershell
$env:GITHUB_TOKEN = "your_github_token"
```

### 3. Bootstrap Flux with GitHub

**For MacOS and Linux:**

```bash
flux bootstrap github \
  --owner=your_github_orga \
  --repository=your_github_project \
  --branch=your_branch \
  --interval=15s \
  --read-write-key \
  --path=clusters/demo/flux
```

**For Windows:**

```powershell
flux bootstrap github `
  --owner=your_github_orga `
  --repository=your_github_project `
  --branch=your_branch `
  --interval=15s `
  --read-write-key `
  --path=clusters/demo/flux
```

The above commands link Flux to a GitHub repository, enabling GitOps-based deployments.

## 🌐 DNS Configuration

**For MacOS and Linux:**

Add the following entries to your local `/etc/hosts` file to resolve sandbox services locally:

```
127.0.0.1 localhost history-server.okdp.sandbox jupyterhub-teama.okdp.sandbox keycloak.okdp.sandbox kad.okdp.sandbox minio.okdp.sandbox minio-console.okdp.sandbox minio-ext.okdp.sandbox okdp-server.okdp.sandbox okdp-ui.okdp.sandbox superset-admins.okdp.sandbox
```

**For Windows:**

Edit the hosts file located at `C:\Windows\System32\drivers\etc\hosts` with administrator privileges and add the following entries:

```
127.0.0.1 localhost history-server.okdp.sandbox jupyterhub-teama.okdp.sandbox keycloak.okdp.sandbox kad.okdp.sandbox minio.okdp.sandbox minio-console.okdp.sandbox minio-ext.okdp.sandbox okdp-server.okdp.sandbox okdp-ui.okdp.sandbox superset-admins.okdp.sandbox
```


## 🔒 Certificate Setup

**For MacOS and Linux:**


Extract the self-signed certificate for secure access to web UIs:

```bash
kubectl get secret default-issuer -n cert-manager -o jsonpath='{.data.ca\.crt}' | base64 -d > okdp-sandbox-selfsigned-ca.crt
```
**For Windows:**
```powershell
$secret = kubectl get secret default-issuer -n cert-manager -o jsonpath="{.data['ca\.crt']}"

[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($secret)) | Out-File -Encoding ascii okdp-sandbox-selfsigned-ca.crt
```


Install this certificate in your system's trust store or browser to validate the OKDP UIs.

## 🔍 Accessing Components

After installation, the following components will be available:

| Component | URL |
|-----------|-----|
| OKDP UI | https://okdp-ui.okdp.sandbox |
| Keycloak | https://keycloak.okdp.sandbox |
| MinIO Console | https://minio-console.okdp.sandbox |
| JupyterHub | https://jupyterhub-teama.okdp.sandbox |
| Superset | https://superset-admins.okdp.sandbox |

## 🔐 Authentication & Credentials

The OKDP Sandbox uses Keycloak as its identity provider. Below are the default credentials for accessing the various components:

### Keycloak

Keycloak is the central authentication service used by all components.

**Admin Access:**
- Username: `admin`
- Password: `admin`

**Pre-configured Users:**

| Username | Password | Roles/Groups |
|----------|----------|--------------|
| usera    | usera    | teama        |
| userb    | userb    | teamb        |
| adm      | adm      | admins       |

### JupyterHub

Authentication is handled through Keycloak OAuth. Use one of the pre-configured users:
- `usera` for regular team access
- `adm` for admin access

### Superset

Superset is configured with Keycloak OAuth:
- Admin access: Use the `adm` user (member of the `admins` group)
- Regular access: Use either `usera` or `userb`

### MinIO Console

For MinIO access, check your deployment configuration for specific credentials. By default, MinIO is configured to use:
- Access Key: Check in the installation logs or the configuration
- Secret Key: Check in the installation logs or the configuration

## 🛠️ Troubleshooting

If you encounter issues during setup:

- **Check cluster status:**
  ```bash
  kind get clusters
  ```

- **Verify Flux installation:**
  ```bash
  flux check
  ```

- **Check pod status:**
  ```bash
  kubectl get pods -A
  ```

- **View logs for specific components:**
  ```bash
  kubectl logs -n <namespace> -l app=<app-name>
  ```

---

<div align="center">
  <sub>Built with ❤️ for the OKDP community</sub>
</div>


