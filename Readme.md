# Kubernetes Dashboard Access via SSH Tunnel

## What This Project Does
Shows how to access Kubernetes Dashboard running on AWS EC2 from your local Windows machine using SSH tunneling. Perfect for remote Kubernetes cluster management.

## The Problem I Solved
- Kubernetes Dashboard runs on `127.0.0.1:8001` (localhost only) on EC2
- Can't access it directly from Windows browser
- Need secure way to connect remotely

## My Solution
Created SSH tunnel to securely connect Windows browser to Kubernetes proxy on EC2.

## Project Architecture
```
Windows Browser (localhost:8081) → SSH Tunnel → EC2 kubectl proxy (127.0.0.1:8001) → Kubernetes Dashboard
```

## What You'll Learn
- Setting up Minikube on AWS EC2
- Creating SSH tunnels for secure remote access
- Accessing Kubernetes Dashboard from anywhere
- Deploying nginx apps with NodePort services

## Technologies Used
- AWS EC2 (Ubuntu 24.04)
- Minikube v1.36.0
- Kubernetes v1.33.1
- Docker v27.5.1
- SSH Tunneling

## Prerequisites
- AWS EC2 instance with Ubuntu
- SSH key pair for EC2 access
- Windows machine with SSH client

## Step-by-Step Setup

### 1. Install Minikube on EC2
```bash
# SSH into your EC2
ssh -i "your-key.pem" ubuntu@your-ec2-ip

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -m 755 kubectl /usr/local/bin/kubectl

# Install Minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
sudo mv minikube /usr/local/bin

# Start cluster
sudo minikube start --vm-driver=none
```

### 2. Start Kubernetes Dashboard
```bash
# Enable dashboard
sudo minikube addons enable dashboard

# Start kubectl proxy
sudo kubectl proxy --address='0.0.0.0' --port=8001 --accept-hosts='^.*$' &
```

### 3. Create SSH Tunnel (Windows)
```cmd
ssh -i "C:\path\to\your\key.pem" -L 8081:127.0.0.1:8001 ubuntu@your-ec2-ip
```

### 4. Access Dashboard
Open browser: `http://localhost:8081/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/`

## Deploy Sample Application

### Create nginx Deployment
```bash
# Create deployment
kubectl create deployment new-dploy --image nginx

# Expose as NodePort
kubectl expose deployment new-dploy --type=NodePort --port=80

# Get NodePort
kubectl get service new-dploy
```

### Access Application
- Via NodePort: `http://your-ec2-ip:nodeport`
- Via kubectl proxy: `http://localhost:8081/api/v1/namespaces/default/services/new-dploy:80/proxy/`

## Key Commands I Used

```bash
# Check cluster status
kubectl cluster-info
minikube status

# View deployments
kubectl get deployment
kubectl get pods -o wide

# Check services
kubectl get service
kubectl describe service service-name

# Troubleshooting
ps aux | grep "kubectl proxy"
netstat -an | grep 8001
```

## What I Learned
- Minikube with `--vm-driver=none` runs directly on host
- SSH tunneling is essential for remote Kubernetes management
- NodePort services need Security Group configuration
- kubectl proxy makes dashboard accessible via simple HTTP

## Troubleshooting

**Dashboard won't load:**
- Check if kubectl proxy is running: `ps aux | grep kubectl`
- Verify SSH tunnel is active on Windows
- Ensure port 8001 is accessible

**Can't access NodePort application:**
- Add NodePort range (30000-32767) to AWS Security Group
- Check service type: `kubectl get service`

## Files in This Repository
- `README.md` - This documentation
- `setup-commands.txt` - All commands used
- `troubleshooting.md` - Common issues and solutions

## Screenshots
(Add screenshots of dashboard and deployed app)

## Next Steps
- Deploy custom Docker images
- Set up ingress controllers
- Implement RBAC for dashboard access
- Add monitoring with Prometheus

## Why This Project Matters
This demonstrates practical DevOps skills:
- Cloud infrastructure (AWS EC2)
- Container orchestration (Kubernetes)
- Network security (SSH tunneling)
- Remote system administration

Perfect for job interviews and DevOps portfolios!

## Contact
- LinkedIn: [Your LinkedIn]
- Email: [Your Email]

---
⭐ Star this repo if it helped you access Kubernetes Dashboard remotely!