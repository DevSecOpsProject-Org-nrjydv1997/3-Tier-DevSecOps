3-Tier DevSecOps Project — Complete Documentation
Project Overview

This project implements a complete DevSecOps pipeline that:

Provisions AWS infrastructure using Terraform

Creates an EKS Kubernetes cluster

Builds CI/CD pipeline using Jenkins

Integrates security scanning tools

Builds and pushes Docker images

Deploys a 3-tier application to Kubernetes

Configures ingress with TLS using cert-manager

Exposes application using a custom domain

Step 1 — Create Installer EC2 VM

This VM is used to:

run Terraform

manage Kubernetes

provision infrastructure

deploy add-ons

Step 2 — Install AWS CLI
sudo apt update
sudo apt install -y unzip curl

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

Verify:

aws --version
Step 3 — Install Terraform
sudo apt update && sudo apt install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt update
sudo apt install terraform -y
Step 4 — Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
Step 5 — Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" -o eksctl.tar.gz
tar -xzf eksctl.tar.gz
sudo mv eksctl /usr/local/bin
Step 6 — Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
Step 7 — Clone Terraform Project
git clone <repo>
cd <repo>
Step 8 — Create EKS Cluster
terraform init
terraform apply
Step 9 — Configure kubeconfig
aws eks update-kubeconfig --name dev-my-cluster --region ap-south-1

Verify:

kubectl get nodes
Step 10 — Add cert-manager module (later)
git pull
terraform init
terraform apply
Step 11 — Create Jenkins VM and SonarQube VM

Two separate VMs were created:

Jenkins Server

SonarQube Server

Step 12 — Install Jenkins
sudo apt update
sudo apt install openjdk-17-jdk -y

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y

Start Jenkins:

sudo systemctl enable jenkins
sudo systemctl start jenkins
Step 13 — Install Docker (Jenkins + SonarQube)
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl restart jenkins
Step 14 — Run SonarQube Container
docker run -d -p 9000:9000 sonarqube:lts-community
Step 15 — Jenkins Plugin Setup

Installed plugins:

Pipeline

Pipeline Stage View

NodeJS

SonarQube Scanner

Docker Pipeline

Kubernetes

Kubernetes CLI

Kubernetes Credentials

Generic Webhook Trigger

Step 16 — Security Tools on Jenkins
Gitleaks
sudo apt install gitleaks
Trivy
sudo apt-get install wget gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
Step 17 — SonarQube Integration

Token created in SonarQube

Added to Jenkins credentials

SonarScanner configured

SonarQube server configured in Jenkins

Step 18 — Docker Credentials

Added DockerHub credentials in Jenkins.

Step 19 — RBAC for Jenkins (prod namespace)

ServiceAccount + Role + ClusterRole configuration created.

Step 20 — Webhook Trigger

Generic webhook configured:

http://JENKINS_IP:8080/generic-webhook-trigger/invoke?token=DevOpsShack123

Triggers pipeline on push to dev branch.

Step 21 — Kubernetes Deployment Files

Includes:

StorageClass

MySQL StatefulSet

Backend Deployment

Frontend Deployment

ClusterIssuer

Ingress

Step 22 — Jenkins Pipeline

Pipeline performs:

Compilation checks

Secret scanning

SonarQube analysis

Trivy scanning

Docker build & push

Kubernetes deployment

Verification

Step 23 — Domain + TLS

Domain:

giftzmania.shop

DNS configured to point to ingress load balancer.

cert-manager issued TLS certificate via Let's Encrypt.

Final Result

Application deployed successfully:

https://giftzmania.shop
