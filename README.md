# DevOps CI/CD Pipeline on AWS

## 📋 Project Overview
A complete DevOps pipeline that automates the build, test, security scanning, and deployment of a Java Spring Boot application on AWS using industry-standard tools.

## 🏗️ Architecture

<img width="3207" height="1903" alt="devops drawio" src="https://github.com/user-attachments/assets/2f88ccee-9e55-4e79-b347-a552a9a435dd" />



## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| AWS EC2 | Cloud Infrastructure |
| Jenkins | CI/CD Automation |
| SonarQube | Code Quality Analysis |
| Nexus | Artifact Repository |
| Docker | Containerization |
| Kubernetes | Container Orchestration |
| Trivy | Security Scanning |
| Prometheus | Metrics Collection |
| Grafana | Monitoring Dashboard |
| Maven | Build Tool |

## ☁️ AWS Infrastructure

### EC2 Instances (t2.medium or t2.large it depends)
- **Jenkins + SonarQube**: CI/CD server
- **Nexus**: Artifact repository  
- **K8s Master**: Kubernetes control plane
- **K8s Slave-1**: Worker node
- **K8s Slave-2**: Worker node
- **Monitor**: Prometheus + Grafana

### Security Group Ports
| Port | Purpose |
|------|---------|
| 22 | SSH access |
| 80 | HTTP |
| 443 | HTTPS |
| 465 | SMTPS (email notifications) |
| 6443 | Kubernetes API server |
| 8080 | Jenkins |
| 8081 | Nexus |
| 9000 | SonarQube |
| 9090 | Prometheus |
| 3000 | Grafana |
| 9115 | Blackbox Exporter |
| 30000-32767 | Kubernetes NodePort |
<img width="714" height="302" alt="لقطة شاشة 2026-03-15 214809" src="https://github.com/user-attachments/assets/70a7fc79-88d8-4a56-b7bb-9993426b26cc" />

## 🔄 Pipeline Stages

1. Git Checkout      → Pull code from GitHub
2. Compile           → mvn compile
3. Test              → mvn test (6 tests)
4. File System Scan  → Trivy FS scan
5. SonarQube         → Code quality analysis
6. Quality Gate      → Verify quality standards
7. Build             → mvn package
8. Publish to Nexus  → Deploy artifact
9. Docker Build      → Build container image
10. Docker Scan      → Trivy image scan
11. Docker Push      → Push to Docker Hub
12. Deploy to K8s    → kubectl apply
13. Verify           → kubectl get pods/svc
14. Email            → Send notification
<img width="1846" height="807" alt="لقطة شاشة 2026-03-12 233021" src="https://github.com/user-attachments/assets/76db2f79-9b23-461b-b85a-b930edbc58a0" />




## 🔐 Security

### RBAC (Role Based Access Control)
```
Role-1: Cluster Admin
Role-2: Good level access  
Role-3: Read only access
```

### Kubernetes RBAC Setup
- ServiceAccount: `jenkins` in namespace `webapps`
- Role: `app-role` with full pod/deployment permissions
- RoleBinding: `app-rolebinding` linking role to jenkins SA
- Secret: `mysecretname` for service account token

### Security Scanning
- **Trivy FS**: Scans filesystem for vulnerabilities
- **Trivy Image**: Scans Docker image before deployment
- **SonarQube**: Detects code bugs and security hotspots

## 📦 Kubernetes Resources

### Service Types Used
- **ClusterIP**: Internal communication between pods
- **NodePort**: External access to application
- **LoadBalancer**: External communication (cloud)

### Deployment
```yaml
Namespace: webapps
Replicas: 2
Image: elansarioumaima/devopsinaws:latest
Port: 8080
NodePort: 30080
```

## 📊 Monitoring

### Prometheus (Port 9090)
- Scrapes metrics every 15 seconds
- Monitors application and infrastructure

### Grafana (Port 3000)
- Visualizes Prometheus metrics
- Blackbox Exporter dashboard
- System level: CPU, RAM monitoring
- Website level: Traffic and availability

### Blackbox Exporter (Port 9115)
- Probes HTTP endpoints
- Checks application availability
  
## 🚀 How to Run
1. Launch EC2 instances on AWS
2. Install tools using provided scripts
3. Configure Jenkins credentials
4. Run the pipeline → Build Now
5. Get the NodePort: `kubectl get svc -n webapps`
6. Access app at `http://[SLAVE_IP]:[NODEPORT]`


## 👩‍💻 Author
**EL ANSARI OUMAIMA**   
DevOps Enthusiast
