# 3-Tier DevSecOps Project (AWS EKS + Jenkins + Security)

This repository demonstrates a **complete end-to-end DevSecOps implementation**, starting from infrastructure provisioning to secure Kubernetes deployment with TLS.

The project shows how CI/CD, security scanning, containerization, and Kubernetes deployment work together in a real workflow.

---

# Project Overview

This project implements a DevSecOps pipeline that:

* Provisions AWS infrastructure using Terraform
* Creates an EKS Kubernetes cluster
* Builds a CI/CD pipeline using Jenkins
* Integrates security scanning tools
* Builds and pushes Docker images
* Deploys a 3-tier application to Kubernetes
* Configures ingress with TLS using cert-manager
* Exposes application using a custom domain

---

# Infrastructure Setup (Installer VM)

## Step 1 — Create Installer EC2 VM

This VM is used to:

* Run Terraform
* Manage Kubernetes
* Provision infrastructure
* Deploy platform add-ons

---

## Step 2 — Install AWS CLI

```bash
sudo apt update
sudo apt install -y unzip curl

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Verify:

```bash
aws --version
```

---

## Step 3 — Install Terraform

```bash
sudo apt update && sudo apt install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update
sudo apt install terraform -y
```

---

## Step 4 — Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

---

## Step 5 — Install eksctl

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" -o eksctl.tar.gz
tar -xzf eksctl.tar.gz
sudo mv eksctl /usr/local/bin
```

---

## Step 6 — Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## Step 7 — Clone Terraform Project

```bash
git clone <repo>
cd <repo>
```

---

## Step 8 — Create EKS Cluster

```bash
terraform init
terraform apply
```

---

## Step 9 — Configure kubeconfig

```bash
aws eks update-kubeconfig --name dev-my-cluster --region ap-south-1
```

Verify:

```bash
kubectl get nodes
```

---

## Step 10 — Add cert-manager module (later)

```bash
git pull
terraform init
terraform apply
```

---

# CI/CD Infrastructure

## Step 11 — Create Jenkins VM and SonarQube VM

Two separate VMs were created:

* Jenkins Server
* SonarQube Server

---

## Step 12 — Install Jenkins

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y
```

Start Jenkins:

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

---

## Step 13 — Install Docker (Jenkins + SonarQube)

```bash
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl restart jenkins
```

---

## Step 14 — Run SonarQube Container

```bash
docker run -d -p 9000:9000 sonarqube:lts-community
```

---

# Jenkins Configuration

## Step 15 — Install Jenkins Plugins

* Pipeline
* Pipeline Stage View
* NodeJS
* SonarQube Scanner
* Docker Pipeline
* Kubernetes
* Kubernetes CLI
* Kubernetes Credentials
* Generic Webhook Trigger

---

## Step 16 — Install Security Tools on Jenkins

### Gitleaks

```bash
sudo apt install gitleaks
```

### Trivy

```bash
sudo apt-get install wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

---

## Step 17 — SonarQube Integration

* Token created in SonarQube
* Added to Jenkins credentials
* SonarScanner configured
* SonarQube server configured in Jenkins

---

## Step 18 — Docker Credentials

DockerHub credentials added in Jenkins.

---

## Step 19 — RBAC for Jenkins (prod namespace)

ServiceAccount, Role, ClusterRole, and bindings created.

---

## Step 20 — Webhook Trigger

```
http://JENKINS_IP:8080/generic-webhook-trigger/invoke?token=DevOpsShack123
```

Triggers pipeline on push to `dev` branch.

---

# Kubernetes Deployment

## Step 21 — Kubernetes Manifests

Includes:

* StorageClass
* MySQL StatefulSet
* Backend Deployment
* Frontend Deployment
* ClusterIssuer
* Ingress

---

## Step 22 — Jenkins Pipeline Responsibilities

Pipeline performs:

* Compilation checks
* Secret scanning (Gitleaks)
* SonarQube analysis
* Trivy scanning
* Docker build & push
* Kubernetes deployment
* Deployment verification

---

# Domain + TLS

## Step 23 — TLS Configuration

Domain:

```
giftzmania.shop
```

DNS points to ingress load balancer.

cert-manager issues TLS certificates using Let’s Encrypt.

---

# Final Result

Application successfully deployed:

```
https://giftzmania.shop
```

---

# Author

DevSecOps Project
