# Docker Super Mario Project - GitOps DevSecOps Pipeline

Infinite Mario in HTML5 JavaScript - using Canvas and Audio elements

This project demonstrates a complete DevSecOps pipeline with GitOps practices using ArgoCD, including SAST scanning with SonarQube, container scanning with Trivy, and Kubernetes deployment with Ingress.

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [CI/CD Pipeline](#cicd-pipeline)
- [Kubernetes Deployment](#kubernetes-deployment)
- [ArgoCD Setup](#argocd-setup)
- [Ingress Configuration](#ingress-configuration)
- [Testing Locally](#testing-locally)
- [Security Scanning](#security-scanning)
- [Troubleshooting](#troubleshooting)

---

## Architecture Overview

```
GitHub Repository
    ↓
GitHub Actions (CI/CD)
    ├── SonarQube SAST Scan
    ├── Docker Build & Push
    └── Trivy Container Scan
    ↓
Docker Hub
    ↓
ArgoCD (GitOps)
    ↓
Azure Kubernetes Service (AKS)
    ├── Pods (Tomcat)
    ├── Service (ClusterIP)
    └── Ingress (NGINX + cert-manager)
```

---

## Prerequisites

- Azure Account with AKS cluster
- Docker Hub account
- GitHub account
- SonarQube server (for SAST scanning)
- kubectl installed locally
- ArgoCD installed on AKS
- Domain name (optional, for production SSL)

---

## Project Structure

```
.
├── .github/workflows/
│   └── gitops.yaml                 # CI/CD pipeline configuration
├── k8s/
│   ├── deployment.yaml             # Kubernetes Deployment
│   ├── service.yaml                # Kubernetes Service (ClusterIP)
│   ├── ingress.yaml                # Ingress with NGINX
│   ├── cert-issuer.yaml            # cert-manager ClusterIssuer
│   ├── docker-secret.yaml          # Docker Hub credentials (template)
│   └── argocd/
│       └── application.yaml        # ArgoCD Application manifest
├── webapp/                         # Super Mario game files
├── Dockerfile                      # Container image definition
├── sonar-project.properties        # SonarQube configuration
└── README.md
```

---

## CI/CD Pipeline

The GitHub Actions pipeline (`.github/workflows/gitops.yaml`) performs the following steps:

### 1. SonarQube SAST Scan
```yaml
- Checkout code
- Run SonarQube analysis
- Check for vulnerabilities and code quality issues
```

### 2. Docker Build & Push
```yaml
- Generate version tag (YYYYMMDD-shortSHA)
- Build Docker image
- Push to Docker Hub with multiple tags:
  - octalcomputer/supermariogitopsproject:YYYYMMDD-shortSHA
  - octalcomputer/supermariogitopsproject:shortSHA
  - octalcomputer/supermariogitopsproject:latest
```

### 3. Trivy Container Scan
```yaml
- Scan Docker image for vulnerabilities
- Generate SARIF report
- Upload results to GitHub Security tab
```

**View scan results**: Repository → Security tab → Code scanning

---

## Kubernetes Deployment

### Docker Image Details
- **Base Image**: Tomcat 9.0.14 on Alpine Linux
- **Exposed Port**: 8080
- **Application Path**: `/usr/local/tomcat/webapps/ROOT/`

### Kubernetes Resources

**Deployment** (`k8s/deployment.yaml`):
- 2 replicas for high availability
- Resource limits: 512Mi memory, 500m CPU
- Health checks (liveness & readiness probes)
- ImagePullSecret for Docker Hub authentication

**Service** (`k8s/service.yaml`):
- Type: ClusterIP (internal)
- Port: 80 → TargetPort: 8080

**Ingress** (`k8s/ingress.yaml`):
- NGINX Ingress Controller
- HTTP/HTTPS support
- Optional TLS with cert-manager

---

## ArgoCD Setup

### 1. Install ArgoCD on AKS
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Access ArgoCD UI
```bash
# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 3. Configure GitHub Repository Access
```bash
# Add repository credentials
argocd repo add https://github.com/YOUR_USERNAME/REPO_NAME.git \
  --username YOUR_GITHUB_USERNAME \
  --password YOUR_GITHUB_PAT
```

### 4. Create Docker Hub Secret
```bash
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=YOUR_DOCKER_USERNAME \
  --docker-password=YOUR_DOCKER_PASSWORD \
  --namespace=default
```

### 5. Deploy ArgoCD Application

**Update** `k8s/argocd/application.yaml`:
- Replace `YOUR_GITHUB_USERNAME` with your GitHub username
- Replace `udemy-gitops-practice-devsecops-sonarqube-sast-scan-supermario-repo` with your repo name

```bash
kubectl apply -f k8s/argocd/application.yaml
```

### 6. Verify Deployment
```bash
# Check application status
argocd app get supermario-gitops

# Check pods
kubectl get pods -l app=supermario

# Check service
kubectl get svc supermario
```

---

## Ingress Configuration

### 1. Install NGINX Ingress Controller
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

### 2. Install cert-manager (for SSL)
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml
```

### 3. Get Ingress External IP
```bash
kubectl get svc -n ingress-nginx ingress-nginx-controller
```

### 4. Configure DNS
Create an **A Record** pointing your domain to the Ingress External IP:
```
supermario.yourdomain.com → INGRESS_EXTERNAL_IP
```

### 5. Update Configuration Files

**Edit** `k8s/cert-issuer.yaml`:
```yaml
email: YOUR_EMAIL@example.com  # Replace with your email
```

**Edit** `k8s/ingress.yaml`:
```yaml
host: supermario.yourdomain.com  # Replace with your domain
```

### 6. Enable TLS (Production)

Uncomment the TLS section in `k8s/ingress.yaml`:
```yaml
spec:
  tls:
  - hosts:
    - supermario.yourdomain.com
    secretName: supermario-tls
```

Change SSL redirect to true:
```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

### 7. Apply Changes
```bash
git add k8s/
git commit -m "Configure Ingress with SSL"
git push
```

ArgoCD will automatically sync and apply the changes.

---

## Testing Locally

### Using /etc/hosts (without real DNS)

1. **Get Ingress External IP**:
   ```bash
   kubectl get svc -n ingress-nginx ingress-nginx-controller
   ```

2. **Edit `/etc/hosts`**:
   ```bash
   sudo nano /etc/hosts
   ```

   Add:
   ```
   INGRESS_EXTERNAL_IP  supermario.yourdomain.com
   ```

3. **Access via HTTP**:
   ```
   http://supermario.yourdomain.com
   ```

4. **Clear Browser Cache** (if needed):
   - Use incognito mode
   - Or manually type `http://` (not `https://`)

---

## Security Scanning

### SonarQube SAST Scan
- **Purpose**: Static Application Security Testing
- **When**: On every push to `main` branch and pull requests
- **Configuration**: `sonar-project.properties`
- **Required Secret**: `SONAR_TOKEN`

### Trivy Container Scan
- **Purpose**: Container vulnerability scanning
- **When**: After Docker image is built
- **Output**: SARIF report uploaded to GitHub Security tab
- **Checks**: OS packages, application dependencies, known CVEs

**View Results**:
```
GitHub Repository → Security tab → Code scanning
```

---

## Troubleshooting

### Pods in CrashLoopBackOff

**Check logs**:
```bash
kubectl logs -l app=supermario --tail=50
kubectl describe pod -l app=supermario
```

**Common Issues**:
- **Port mismatch**: Ensure container port 8080 matches Dockerfile
- **OOMKilled**: Increase memory limits in deployment.yaml
- **ImagePullBackOff**: Create dockerhub-secret for authentication

### ArgoCD Not Syncing

**Check application status**:
```bash
argocd app get supermario-gitops
```

**Manual sync**:
```bash
argocd app sync supermario-gitops
```

**Check repository access**:
```bash
argocd repo list
```

### Ingress 404 Not Found

**Check Ingress status**:
```bash
kubectl get ingress
kubectl describe ingress supermario-ingress
```

**Verify NGINX controller**:
```bash
kubectl get pods -n ingress-nginx
```

**Check Service endpoints**:
```bash
kubectl get endpoints supermario
```

### SSL Certificate Issues

**Check certificate status**:
```bash
kubectl get certificate
kubectl describe certificate supermario-tls
```

**Check cert-manager logs**:
```bash
kubectl logs -n cert-manager -l app=cert-manager
```

**For testing**: Disable TLS in `k8s/ingress.yaml` and use HTTP

---

## GitHub Secrets Required

Configure these secrets in **GitHub Repository → Settings → Secrets and variables → Actions**:

- `SONAR_TOKEN`: SonarQube authentication token
- `DOCKER_USERNAME`: Docker Hub username
- `DOCKER_PASSWORD`: Docker Hub password or access token

---

## Accessing the Application

### Production (with DNS):
```
https://supermario.yourdomain.com
```

### Testing (with /etc/hosts):
```
http://supermario.yourdomain.com
```

### Direct Service (LoadBalancer - deprecated):
```
http://SERVICE_EXTERNAL_IP
```

---

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test locally
5. Submit a pull request

---

## License

This project is for educational purposes as part of DevSecOps and GitOps learning.

---

## Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [cert-manager Documentation](https://cert-manager.io/docs/)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [SonarQube Documentation](https://docs.sonarqube.org/)

---

## Acknowledgments

- Original Super Mario game ported from Notch's Java version
- Modified for DevSecOps pipeline learning
- Container security enhanced with Trivy scanning
- GitOps practices implemented with ArgoCD
