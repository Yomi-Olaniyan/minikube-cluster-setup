# Minikube Kubernetes Cluster Setup & Application Deployment

## 🎯 Project Overview
Complete guide for setting up a production-ready Minikube cluster on AWS EC2, deploying containerized applications, and implementing secure remote access through SSH tunneling. This project demonstrates end-to-end Kubernetes orchestration with custom Docker images and external accessibility.

## 🏗️ Architecture
```
┌─────────────────┐    SSH Tunnel    ┌─────────────────┐
│   Local Machine │◄────────────────►│  AWS EC2 (Ubuntu)│
│   (Windows)     │    Port 8081     │                 │
└─────────────────┘                  └─────────────────┘
                                              │
                                     ┌─────────────────┐
                                     │   Minikube      │
                                     │   Cluster       │
                                     │                 │
                                     │ ┌─────────────┐ │
                                     │ │Dashboard    │ │
                                     │ │NodePort App │ │
                                     │ │Custom nginx │ │
                                     │ └─────────────┘ │
                                     └─────────────────┘
```

## 🛠️ Technologies Used
- **Kubernetes**: v1.33.1 (Minikube)
- **Docker**: v27.5.1
- **Platform**: AWS EC2 Ubuntu 24.04.2 LTS
- **Container Registry**: Docker Hub
- **Networking**: SSH Tunneling, NodePort Services
- **Tools**: kubectl, minikube, Docker CLI

## 🚀 Installation & Setup

### Prerequisites
- AWS EC2 instance (Ubuntu 24.04+)
- SSH key pair for EC2 access
- Docker installed on EC2 instance
- Security Group configured for ports 22, 80, and NodePort range (30000-32767)

### 1. kubectl Installation
```bash
# Download latest kubectl binary
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Install kubectl with proper permissions
sudo install -m 755 kubectl /usr/local/bin/kubectl

# Verify installation
kubectl version --client

# Cleanup
rm kubectl
```

### 2. Minikube Installation

#### Method A: Direct Download (Recommended)
```bash
# Download minikube binary
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Make executable and move to PATH
chmod +x minikube
sudo mv minikube /usr/local/bin

# Verify installation
which minikube
minikube version
```

#### Method B: Alternative Download
```bash
# Alternative download method
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64

# Install and cleanup
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
```

### 3. Cluster Initialization
```bash
# Switch to root user for Docker driver
sudo su

# Start Minikube cluster with Docker driver
minikube start --vm-driver=none

# Verify cluster status
minikube status
```

**Expected Output:**
```
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

## 🔧 Cluster Configuration & Verification

### System Information Commands
```bash
# Check Linux distribution
cat /etc/os-release

# View cluster information
kubectl cluster-info

# Get cluster nodes with detailed info
kubectl get nodes -o wide

# Check Minikube container (if using Docker driver)
docker ps
docker exec -it minikube bash
```

### Kubernetes Components Verification
```bash
# View all system pods
kubectl -n kube-system get pods
```

**Expected System Pods:**
- `coredns` - DNS resolution
- `etcd` - Key-value store
- `kube-apiserver` - API server
- `kube-controller-manager` - Controller manager
- `kube-proxy` - Network proxy
- `kube-scheduler` - Pod scheduler
- `storage-provisioner` - Storage management

## 📊 Dashboard Setup & Access

### Enable Kubernetes Dashboard
```bash
# List available addons
minikube addons list

# Enable dashboard addon
minikube addons enable dashboard

# Get dashboard URL
minikube dashboard --url
```

### Secure Remote Access via SSH Tunnel

#### Start kubectl Proxy
```bash
# Start proxy server on cluster
kubectl proxy --address='0.0.0.0' --port=8001 --disable-filter=true &

# Alternative: Standard proxy (localhost only)
kubectl proxy
```

#### Create SSH Tunnel (Windows)
```cmd
# From Windows Command Prompt
ssh -i "C:\Users\Master\Downloads\olaabKP.pem" -L 8081:127.0.0.1:8001 ubuntu@54.164.82.121

# Verify tunnel is active
netstat -an | findstr 8081
```

#### Access Dashboard
- **Local URL**: `http://localhost:8081/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/`
- **Direct URL**: `http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/`

### Network Troubleshooting
```bash
# Check kubectl proxy port
netstat -tlnp | grep kubectl | tail -1

# Save network state for comparison
netstat -tlnp | grep kubectl > before.txt

# Test dashboard connectivity
curl -v http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

## 🚢 Application Deployment

### 1. Deploy Standard nginx Application
```bash
# Create deployment
kubectl create deployment yolaab --image nginx

# Expose deployment as service
kubectl expose deployment yolaab --port 80 --type=NodePort

# Get service details
kubectl get service yolaab
```

### 2. Service Configuration
```bash
# If service type is ClusterIP, change to NodePort
kubectl edit service yolaab

# Verify service configuration
kubectl describe service yolaab
```

**Example Service Output:**
```
NAME      TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
yolaab    NodePort   10.101.3.198   <none>        80:31772/TCP   45m
```

## 🐳 Custom Docker Image Pipeline

### Project Structure
```
kube-nginx/
├── Dockerfile
├── index.html
└── README.md
```

### 1. Create Custom Application
```bash
# Create project directory
mkdir kube-nginx && cd kube-nginx

# Create Dockerfile
cat > Dockerfile << EOF
FROM nginx:latest
COPY index.html /usr/share/nginx/html/
EXPOSE 80
EOF

# Create custom HTML page
cat > index.html << EOF
<!DOCTYPE html>
<html>
<head>
    <title>OLAAB DevOps Portfolio</title>
</head>
<body>
    <h1>Welcome to OLAAB DevOps Page!</h1>
    <p>This custom deployment, running on Kubernetes, is created by OLAAB!</p>
    <p>Think DEVOPS! Think CLOUD! Think AUTOMATION!</p>
</body>
</html>
EOF
```

### 2. Docker Registry Workflow
```bash
# Login to Docker Hub
docker login -u olaab

# Build custom image
docker build -t 8th-august-image .

# Tag image for registry
docker tag 8th-august-image olaab/minikube-orchestration:v1.2

# Push to Docker Hub
docker push olaab/minikube-orchestration:v1.2

# Update Kubernetes deployment
kubectl set image deployment/yolaab nginx=olaab/minikube-orchestration:v1.2
```

### 3. Verify Deployment
```bash
# Check rollout status
kubectl rollout status deployment/yolaab

# Verify pods are running
kubectl get pods -l app=yolaab

# Get service endpoint
kubectl get service yolaab
```

## 🌐 External Access & Testing

### Access Methods
1. **NodePort Access**: `http://<EC2-PUBLIC-IP>:<NODEPORT>`
2. **Example**: `http://44.201.249.7:31772`
3. **Port Forward**: `kubectl port-forward service/yolaab 8080:80`

### Security Group Configuration
Ensure AWS Security Group allows:
- **SSH**: Port 22
- **HTTP**: Port 80
- **NodePort Range**: 30000-32767
- **Custom Port**: The specific NodePort (e.g., 31772)

### Testing Commands
```bash
# Test from within cluster
kubectl run test-pod --image=busybox -it --rm -- wget -qO- http://yolaab.default.svc.cluster.local

# Test NodePort locally
curl http://localhost:31772

# Check node internal/external IPs
kubectl get nodes -o wide
```

## 🔍 Monitoring & Troubleshooting

### Common Commands
```bash
# View deployment status
kubectl get deployments
kubectl describe deployment yolaab

# Check pod logs
kubectl logs -f deployment/yolaab

# Debug service issues
kubectl get endpoints yolaab
kubectl describe service yolaab

# View cluster events
kubectl get events --sort-by='.lastTimestamp'
```

### Common Issues & Solutions

| Issue | Symptoms | Solution |
|-------|----------|----------|
| NodePort not accessible | Connection timeout | Check Security Group rules |
| Pod CrashLoopBackOff | Pod restart loops | Check `kubectl logs` and resource limits |
| Service ClusterIP only | Can't access externally | Change service type to NodePort |
| Dashboard not loading | 404 or connection error | Verify SSH tunnel and proxy status |

### Resource Management
```bash
# Check resource usage
kubectl top nodes
kubectl top pods

# Scale deployment
kubectl scale deployment yolaab --replicas=3

# Delete resources (cleanup)
kubectl delete deployment yolaab
kubectl delete service yolaab
minikube delete
```

## 📈 Performance Metrics
- **Cluster Start Time**: ~2-3 minutes
- **Deployment Time**: ~30 seconds
- **Image Pull Time**: ~1-2 minutes (first time)
- **Service Ready Time**: ~10 seconds
- **Dashboard Access**: Immediate via SSH tunnel

## 🔐 Security Best Practices
1. **SSH Key Management**: Use strong key pairs for EC2 access
2. **Security Groups**: Restrict access to necessary ports only
3. **Network Policies**: Implement pod-to-pod communication rules
4. **RBAC**: Configure role-based access control
5. **Secrets Management**: Use Kubernetes secrets for sensitive data

## 🧹 Cleanup Commands
```bash
# Stop specific deployments
kubectl delete deployment yolaab
kubectl delete service yolaab

# Remove Minikube cluster completely
minikube delete

# Clean Docker images
docker rmi olaab/minikube-orchestration:v1.2
docker system prune -a
```

## 📝 Key Learning Outcomes
- ✅ Minikube cluster setup and management
- ✅ Kubernetes service types and networking
- ✅ Docker image creation and registry operations
- ✅ SSH tunneling for secure remote access
- ✅ AWS EC2 and Security Group configuration
- ✅ Custom application deployment and scaling
- ✅ Troubleshooting common Kubernetes issues

## 🤝 Contributing
Feel free to fork this repository and submit pull requests for improvements. Please ensure all commands are tested before submission.

## 📄 License
MIT License - Feel free to use this guide for educational and professional purposes.

## 🙏 Acknowledgments
- Kubernetes community for excellent documentation
- Minikube project for simplified local development
- Docker Hub for reliable container registry services