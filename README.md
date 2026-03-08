# K3s Infrastructure + CI/CD Deployment Project

This project demonstrates a complete **DevOps workflow** including:

-   Infrastructure provisioning using Terraform
-   Kubernetes cluster setup using K3s
-   Containerization using Docker
-   CI/CD automation using GitHub Actions
-   Cloud infrastructure on AWS

The pipeline automatically:

1.  Creates a K3s server
2.  Builds Docker images
3.  Pushes images to Docker Hub
4.  Deploys the application into Kubernetes

------------------------------------------------------------------------

# Project Structure

    .
    ├── .github
    │   └── workflows
    │
    ├── k3s_infra_create
    │   ├── backend.tf
    │   ├── main.tf
    │   ├── outputs.tf
    │   ├── provider.tf
    │   └── variables.tf
    │
    ├── k3s-manifest
    │   ├── deploy.yml
    │   ├── ingress.yml
    │   └── svc.yml
    │
    ├── Dockerfile
    └── index.html

------------------------------------------------------------------------

# Prerequisites

Install the following tools on your local machine or server.

## Install Git

Ubuntu

    sudo apt update
    sudo apt install git -y

Verify

    git --version

------------------------------------------------------------------------

## Install Terraform

    sudo apt update
    sudo apt install -y gnupg software-properties-common curl
    curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
    sudo apt-add-repository "deb https://apt.releases.hashicorp.com $(lsb_release -cs) main"
    sudo apt update
    sudo apt install terraform

Verify

    terraform --version

------------------------------------------------------------------------

## Install AWS CLI

    sudo apt install awscli -y

Verify

    aws --version

------------------------------------------------------------------------

# Configure AWS CLI

Create an IAM user in AWS with permissions:

-   EC2 Full Access
-   Security Group creation
-   S3 access
-   DynamoDB access

Run:

    aws configure

Verify:

    aws configure list

------------------------------------------------------------------------

# Clone the Repository

    git clone https://github.com/YOUR_USERNAME/k3s_project.git
    cd k3s_project

------------------------------------------------------------------------

# Create K3s Infrastructure

    cd k3s_infra_create
    terraform init
    terraform plan
    terraform apply --auto-approve

After completion Terraform will output the **Public IP** of the K3s
server.

------------------------------------------------------------------------

# Connect to the K3s Server

    ssh -i your-key.pem ubuntu@PUBLIC_IP

Verify Kubernetes:

    sudo kubectl get nodes

Verify Docker:

    docker --version

------------------------------------------------------------------------

# Setup GitHub Self Hosted Runner

    sudo mkdir K3s_Project
    cd K3s_Project

Go to GitHub repository:

Settings → Actions → Runners → New Self Hosted Runner

Select **Linux** and follow the commands shown.

Runner status should show:

    Idle

------------------------------------------------------------------------

# Configure GitHub Secrets

Go to:

Settings → Secrets and Variables → Actions → New Repository Secret

Create the following:

  DOCKER_USERNAME   DockerHub username
  DOCKER_PASSWORD   DockerHub access token
  IMAGE_NAME        Docker image repository

Example:

    IMAGE_NAME=username/k3s-html-app

------------------------------------------------------------------------

# GitHub Workflow File

Create:

    .github/workflows/ci-cd.yml

Example:

``` yaml
name: Build and Deploy Application

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: self-hosted

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Docker Hub Login
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login           -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Build Docker Image
        run: |
          docker build -t ${{ secrets.IMAGE_NAME }}:${{ github.sha }} .

      - name: Push Docker Image
        run: |
          docker push ${{ secrets.IMAGE_NAME }}:${{ github.sha }}

      - name: Update Kubernetes Deployment
        run: |
          kubectl set image deployment/html-app           html-app=${{ secrets.IMAGE_NAME }}:${{ github.sha }}
```

------------------------------------------------------------------------

# Deploy Kubernetes Manifests

    cd k3s-manifest
    kubectl apply -f .

This will create:

-   Deployment
-   Service
-   Ingress

------------------------------------------------------------------------

# Access Application

Open:

    http://K3S_PUBLIC_IP:80

------------------------------------------------------------------------

# CI/CD Flow

Developer pushes code:

    git push origin main

Pipeline automatically:

1.  Builds Docker image
2.  Tags image with commit ID
3.  Pushes image to DockerHub
4.  Updates Kubernetes deployment
5.  Deploys latest version

------------------------------------------------------------------------

# Technologies Used

Terraform\
Docker\
K3s Kubernetes\
GitHub Actions\
AWS Cloud
