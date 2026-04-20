# GitOps with Argo CD

## Overview

This project demonstrates **GitOps** principles using **Argo CD** - a declarative, GitOps continuous delivery tool for Kubernetes.

Argo CD automates the deployment of applications to Kubernetes clusters by continuously monitoring Git repositories and ensuring the live state matches the desired state defined in Git (the single source of truth).

---

## Architecture Diagram

![Argo CD Architecture](/Day-2/screenshot/image1.png)

*Architecture diagram showing Argo CD components*

---

## Component Details

### 1. Application Controller
The **Application Controller** is the brain of Argo CD. It is responsible for:

- Continuously monitoring all Argo CD `Application` resources
- Comparing the live state (what's currently running in the cluster) with the desired state (defined in Git)
- Detecting configuration drift
- Reconciling differences by triggering sync operations
- Managing the lifecycle of applications (Health checks, Sync status, History, Rollbacks)

It runs as a Kubernetes Deployment and uses a workqueue to process applications efficiently. It is highly scalable and can handle thousands of applications.

### 2. Repo Server
The **Repo Server** is responsible for fetching and processing manifests from Git repositories.

Key responsibilities:
- Cloning Git repositories
- Generating Kubernetes manifests from various sources:
  - Plain YAML/JSON
  - Helm charts
  - Kustomize
  - Custom plugins
- Caching repository content for performance
- Performing security scanning and template rendering

It runs as a stateless deployment and can be horizontally scaled based on load.

### 3. Redis
**Redis** serves as the caching and pub/sub layer for Argo CD.

Usage:
- Caches repository contents (cloned repos, generated manifests)
- Stores short-lived data and session information
- Acts as a message broker for internal communication between components
- Improves performance by reducing repeated Git operations

It is deployed as a StatefulSet (usually) with persistence for better reliability.

### 4. API Server
The **API Server** is the front-end of Argo CD. It provides:

- gRPC and REST APIs for all operations
- Authentication and authorization
- Web UI (UI server)
- Webhook endpoints (for Git events, CI integrations)
- RBAC enforcement
- Integration with external identity providers via Dex

This is the component that the Argo CD CLI (`argocd`), Web UI, and external systems interact with.

### 5. Dex Server
**Dex** is an OpenID Connect (OIDC) identity service that provides authentication.

Features:
- Integrates with external identity providers (LDAP, GitHub, Google, Okta, Microsoft, SAML, etc.)
- Issues short-lived tokens to the Argo CD API Server
- Handles SSO (Single Sign-On)
- Maps external user/group identities to Argo CD RBAC policies

Dex runs as a separate deployment and is optional if using local users or other auth methods.

---

## How Argo CD Works (GitOps Flow)

1. Developers commit infrastructure/application changes to Git
2. Argo CD's **Repo Server** detects changes and generates manifests
3. **Application Controller** compares desired state (Git) vs live state (cluster)
4. If drift is found or a manual sync is triggered, the controller applies changes
5. **API Server** exposes status, logs, and allows manual intervention via UI/CLI
6. **Redis** ensures high performance and caching
7. **Dex** handles secure user authentication

This creates a fully declarative, auditable, and automated deployment pipeline.

---

## Key Benefits of This Setup

- **Single Source of Truth**: Everything lives in Git
- **Automated Sync**: Continuous reconciliation
- **Audit Trail**: All changes are tracked via Git history
- **Multi-tenancy & RBAC**: Fine-grained access control
- **Multi-cluster Support**: Manage hundreds of clusters from one control plane
- **Extensibility**: Support for Helm, Kustomize, and custom plugins

---
