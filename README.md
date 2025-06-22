# MessageBoardk8sRepo

# 🚀 GitOps-Based Kubernetes Deployment with Github Actions, Argo CD, Kustomize & Sealed Secrets

This repository manages the Kubernetes manifests and deployment pipeline for the application across `dev`, `staging`, and `prod` environments using:
---

- ✅ **GitHub Actions** for CI/CD
- 🧩 **Kustomize** for environment-specific configurations
- 🚀 **Argo CD** for continuous deployment
- 🔐 **Sealed Secrets** for secure secret management
- 📡 **Prometheus + Alertmanager** for monitoring and alerting


---

### ✅ Prerequisites
- Kubernetes cluster (EKS, GKE, Minikube, etc.) - v1.19+
- Helm (v3.2.0+)
- kubectl configured with cluster access
- Tools: kubectl, kustomize, kubeseal, argocd
- Sealed Secrets controller installed in the cluster
- Argo CD installed and configured to track this repository


---

## 📁 Repository Structure

```bash
.
├── base/
│   ├── deployment.yaml           # Core Deployment spec
│   ├── kustomization.yaml        # Base kustomization file
│   ├── service.yaml              # Main service definition
│   ├── service-monitor.yaml     # Additional service (e.g., for monitoring)
│
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   ├── patch-deployment.yaml
│   │   ├── patch-secret.yaml
│   │   ├── patch-service.yaml
│   │   ├── patch-prometheus.yaml
│   │   └── namespace.yaml
│   ├── staging/
│   │   └── ...
│   └── prod/
│       └── ...
│
│── argo/
    ├── dev-app.yaml              # Argo CD application for dev
    ├── staging-app.yaml          # Argo CD application for staging
    └── prod-app.yaml             # Argo CD application for prod

- `base/` contains shared configurations.
- `overlays/` contains environment-specific patches and settings.


# 🚀 CI/CD Pipeline for Java Spring Boot with GitHub Actions

🔧 This project includes a GitHub Actions-based CI/CD pipeline that:

🧾 Checks out code

☕ Builds Java 17 Spring Boot app with Maven

🐳 Builds and pushes Docker image to Docker Hub

🖊 Updates the image tag in the corresponding Kustomize overlay

🔁 Commits the updated kustomization.yaml back to the repo

🚀 Triggers deployment in Argo CD

---

## 📂 Workflow File

The GitHub Actions workflow is defined in:
.github/workflows/docker-build-push.yml

---

## 🧰 Prerequisites

Before using this workflow, make sure the following are set up:

- A valid `Dockerfile` in the project root (for building your Spring Boot app)
- A [Docker Hub account](https://hub.docker.com/)
- GitHub repository secrets:

| Secret Name           | Description                                |
|-----------------------|--------------------------------------------|
| `DOCKERHUB_USERNAME`  | Your Docker Hub username                   |
| `DOCKERHUB_TOKEN`     | Docker Hub [access token](https://hub.docker.com/settings/security) |
| `ARGOCD_SERVER`       | *(Optional)* Argo CD server URL (e.g. https://argocd.example.com) |
| `ARGOCD_AUTH_TOKEN`   | *(Optional)* Argo CD API token             |

---

## ⚙️ What the Pipeline Does

### On Push to `main` or `release/*` Branches:

- 🛠️ Checks out code
- ☕ Sets up Java 17 (Temurin)
- 📦 Builds the project using Maven
- 🐳 Builds a Docker image
- 🔖 Tags the image as:
  - `latest`
  - Git short SHA (e.g., `a1b2c3d`)
- 🚀 Pushes the image to Docker Hub

### Optional Argo CD Sync

If configured, it can also trigger a sync of your Argo CD application after pushing the new image.

---

## 📝 Dockerfile Example (Spring Boot)

Here’s the Dockerfile:

```dockerfile
FROM eclipse-temurin:17-jdk-jammy
WORKDIR /app
COPY target/webapp-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]


# Kubernetes Monitoring Stack (Prometheus + Alertmanager)

![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white)
![Alertmanager](https://img.shields.io/badge/Alertmanager-FFA500?style=for-the-badge)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=Helm&logoColor=white)

Automated monitoring solution for Kubernetes clusters with built-in CI/CD integration practices.


### Key DevOps Focus Areas Included:
1. **CI/CD Integration** - Used Github Actions for pipeline automation
2. **GitOps Ready**: Designed for ArgoCD integration
3. **Automated Alerting**: Pre-configured alert rules with severity levels
4. **Self-Healing**: Built-in alert resolution tracking
5. **Scalable**: HA-ready Alertmanager configuration

## Deployment

---

## 🛡 CI/CD Integration

You can include Git push automation in CI/CD tools like GitHub Actions:

```yaml
      - name: Determine overlay path
        id: overlay
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "OVERLAY=prod" >> "$GITHUB_ENV"
          elif [[ "${{ github.ref }}" == "refs/heads/staging" ]]; then
            echo "OVERLAY=staging" >> "$GITHUB_ENV"
          elif [[ "${{ github.ref }}" == "refs/heads/dev" ]]; then
            echo "OVERLAY=dev" >> "$GITHUB_ENV"
          else
            echo "Unsupported branch: ${{ github.ref }}"
            exit 1
          fi

      - name: Update image tag in Kustomization
        run: |
          cd k8s-repo/kustomize/overlays/${{ env.OVERLAY }}
          kustomize edit set image gitauwairimu/messageboard=gitauwairimu/messageboard:${{ github.sha }}

      - name: Commit and push changes
        run: |
          cd k8s-repo
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add kustomize/overlays/${{ env.OVERLAY }}/kustomization.yaml
          git commit -m "Update image for ${{ env.OVERLAY }} to ${{ github.sha }}"
          git push


## ⚙️ Argo CD Application Manifest

The above YAML defines an Argo CD Application that:
- Watches the `dev` overlay in this Git repo.
- Applies changes to the `messageboard-dev` namespace.
- Auto-syncs when changes are pushed to the repo.


# dev-argo-app.yaml
```yaml
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: messageboard-dev
      namespace: argocd
    # data:
    #   application.resync.period: 60s
    spec:
      project: default
      source:
        repoURL: https://github.com/gitauwairimu/MessageBoardk8sRepo
        targetRevision: main
        path: kustomize/overlays/dev
      destination:
        server: https://kubernetes.default.svc
        namespace: messageboard-dev
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
      ignoreDifferences:
      - group: bitnami.com
        kind: SealedSecret
        jsonPointers:
        - /status
      - group: monitoring.coreos.com
        kind: Prometheus
        jsonPointers:
        - /status



## 🔁 Sync Behavior

This setup uses **auto-sync** with:
- `prune: true` – Removes Kubernetes resources not found in Git.
- `selfHeal: true` – Fixes out-of-sync resources automatically.

To trigger manual sync (if auto-sync is disabled):

```bash
argocd app sync my-app



---


## 🛠 Update Workflow

1. Make changes to your YAML or Kustomize files.
2. Commit and push to the Git repo.
3. Argo CD detects the change and applies it to the cluster automatically.

No need to manually `kubectl apply`.


📡 Monitoring & Alerting
This setup includes a monitoring stack with Prometheus and Alertmanager.

Features:
🔔 Pre-configured alerts with severity levels

🛡️ Self-healing notifications

📈 Custom ServiceMonitor support

☸️ Integrated with CI/CD pipelines