# AWS EKS Deployment Guide

## Overview
This guide provides step-by-step instructions for deploying applications on Amazon Elastic Kubernetes Service (EKS).

## Project Structure
```
.
├── 
│   ├── app.py               # Python application
│   ├── Dockerfile          # Docker configuration
│   └── requirements.txt    # Python dependencies
├── 
│   ├── deployment.yml      # Kubernetes deployment configuration
│   └── service.yml         # Kubernetes service configuration
└── 
    └── cluster.yml         # EKS cluster configuration
```

## Prerequisites
- AWS CLI v2
- kubectl (Kubernetes CLI)
- eksctl (EKS CLI)
- AWS Account with administrative access
- AWS IAM user with appropriate permissions
- Docker installed on your local machine

## Step-by-Step Guide

### 1. Install and Configure Prerequisites

#### AWS CLI Installation
```bash
# Download and install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify installation
aws --version
```

#### Create IAM User and Configure AWS
1. Create an IAM user with appropriate permissions in AWS Console
2. Generate access key for the IAM user
3. Configure AWS CLI:
```bash
aws configure
# Enter your AWS Access Key ID
# Enter your AWS Secret Access Key
# Enter your preferred region (e.g., us-east-1)
# Enter your preferred output format (json recommended)
```

#### Install kubectl
```bash
# Download kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Change permissions
chmod +x kubectl

# Move to system path
sudo mv kubectl /usr/local/bin/

# Verify installation
kubectl version --client
```

#### Install eksctl
```bash
# Download eksctl
curl -LO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz"

# Extract and install
sudo tar -xzf eksctl_$(uname -s)_amd64.tar.gz -C /usr/local/bin

# Verify installation
eksctl version
```

#### Docker Setup
```bash
# Install Docker (Ubuntu)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify Docker installation
docker --version

# (Optional) Add your user to docker group to run Docker without sudo
sudo usermod -aG docker $USER
# Note: You'll need to log out and back in for this to take effect
```

For other operating systems, visit:
- Ubuntu/Debian: https://docs.docker.com/engine/install/ubuntu/
- CentOS: https://docs.docker.com/engine/install/centos/
- Windows: https://docs.docker.com/desktop/install/windows-install/
- macOS: https://docs.docker.com/desktop/install/mac-install/

### 2. Create EKS Cluster

#### Create cluster configuration
```bash
vi cluster.yml
```

#### Deploy cluster
```bash
# Create cluster
eksctl create cluster -f cluster.yml

# Verify cluster creation
Replace `<region>` with your actual region.
eksctl get cluster --region <region>

# Update kubeconfig
Replace `<region>` with your actual region.
aws eks --region <region> update-kubeconfig --name my-eks-cluster

# Verify nodes
kubectl get nodes
```

### 3. Prepare Application

#### Create Application Files

```bash
vi app.py
```
#### Create requirements.txt
```bash
vi requirements.txt
```


#### Create Dockerfile
```bash
vi Dockerfile
```

### 4. Build and Push Docker Image
Replace `<aws_account_id>` and `<region>` with your actual AWS account ID and region.

```bash
# Build image
docker build -t my-app .

# Tag image
docker tag my-app:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-app:latest

# Authenticate to AWS ECR
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com

# Create ECR repository
aws ecr create-repository --repository-name my-app

# Push image
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-app:latest
```

### 5. Deploy Application to EKS

#### Create Kubernetes configurations
```bash
vi kubernetes/deployment.yml
```

#### Create service.yml
```bash
vi kubernetes/service.yml
```


#### Deploy application
```bash
# Apply configurations
kubectl apply -f deployment.yml
kubectl apply -f service.yml

# Verify deployment
kubectl get pods
kubectl get service my-app-service
```
Get the public IP of the service and open it in your browser.
### 6. Cleanup

```bash
# Delete Kubernetes resources
kubectl delete -f deployment.yml
kubectl delete -f service.yml

# Delete your resources from AWS Managmenet Console
- Delete your EKS cluster from EKS Console
- Delete your ECR repository from ECR Console
```

## Troubleshooting
- If pods are not running, check logs: `kubectl logs <pod-name>`
- For service issues, verify service status: `kubectl describe service my-app-service`
- For cluster issues, check CloudFormation in AWS Console

## Additional Resources
- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/)
- [kubectl Documentation](https://kubernetes.io/docs/reference/kubectl/)
- [eksctl Documentation](https://eksctl.io/)

    
        
