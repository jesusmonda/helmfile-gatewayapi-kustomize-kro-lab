# EKS Bootstrap with ArgoCD, KRO & Gateway API

Laboratory demonstrating professional EKS cluster bootstrap with GitOps, custom operators, and shared infrastructure patterns.

## 🎯 What This Lab Demonstrates

- **EKS Bootstrap**: Automated cluster setup with essential controllers
- **GitOps with ArgoCD**: Declarative infrastructure management
- **KRO (Kubernetes Resource Operator)**: Standardized application deployments
- **Shared Gateway API**: Cost-efficient single ALB for multiple services
- **Multi-Repository Pattern**: Separation of infrastructure, applications, and code

## 🏗️ Multi-Repository Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Repository: k8s-infrastructure            │
│                                                              │
│  ┌──────────────┐         ┌─────────────────────┐          │
│  │   ArgoCD     │────────▶│    Controllers      │          │
│  │ (App of Apps)│         │  - ALB, EBS, EFS    │          │
│  └──────────────┘         │  - KRO, Gateway API │          │
│                           └─────────────────────┘          │
│                                     │                        │
│                           ┌─────────▼─────────┐            │
│                           │  Shared Gateway   │            │
│                           │  (Single ALB)     │            │
│                           └─────────┬─────────┘            │
└─────────────────────────────────────┼──────────────────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    │                                   │
┌───────────────────▼──────────────┐  ┌────────────────▼──────────────┐
│ Repository: k8s-applications     │  │ Repository: mondamail         │
│                                  │  │                               │
│  ┌────────────────────────┐     │  │  ┌──────────────────────┐    │
│  │ mondamail/             │     │  │  │ src/                 │    │
│  │  └─ application.yaml   │     │  │  │ Dockerfile           │    │
│  │     (KRO Resource)     │     │  │  │ deployment.yaml      │    │
│  │                        │     │  │  │  (KRO Config)        │    │
│  │ service-2/             │     │  │  │   - replicas: 2      │    │
│  │  └─ application.yaml   │     │  │  │   - env vars         │    │
│  └────────────────────────┘     │  │  │   - resources        │    │
│                                  │  │  └──────────────────────┘    │
└──────────────────────────────────┘  └────────────────────────────┘
```

## 🚀 Bootstrap Process

### Step 1: Bootstrap ArgoCD

```bash
cd bootstrap/initial
make deploy ENV=dev
```

This installs:
1. ArgoCD via Helmfile
2. Configures ArgoCD namespace
3. Deploys App of Apps pattern

### Step 2: ArgoCD Deploys Controllers

ArgoCD automatically syncs and installs:
- AWS Load Balancer Controller
- Gateway API CRDs
- KRO Operator

### Step 3: ArgoCD Deploys Shared Resources

- Shared Gateway (single ALB)
- Manual CRDs
- Base configurations

### Step 4: Deploy Applications

In `k8s-applications` repository:

```yaml
# mondamail/application.yaml
apiVersion: kro.run/v1alpha1
kind: Mondamail
metadata:
  name: mondamail
  namespace: production
spec:
  deployment:
    replicas: 3
    image: ghcr.io/org/mondamail:v1.2.0
    env:
      - name: DATABASE_URL
        value: "postgres://..."
  resources:
    requests:
      cpu: "200m"
      memory: "256Mi"
```

### Repository Separation

- **k8s-infrastructure**: Platform team manages
- **k8s-applications**: Platform team + App teams
- **mondamail**: Developers own completely