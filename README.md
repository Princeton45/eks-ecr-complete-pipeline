# Complete CI/CD Pipeline with EKS and AWS ECR

This repository contains my implementation of a complete CI/CD pipeline using AWS EKS and ECR, demonstrating automated building and deployment of a Java application to Kubernetes.

![diagram](https://github.com/Princeton45/eks-ecr-complete-pipeline/blob/main/images/diagram.png)

## 🛠 Technologies Used

- AWS EKS (Elastic Kubernetes Service)
- AWS ECR (Elastic Container Registry)
- Jenkins
- Docker
- Kubernetes
- Java
- Maven
- Git
- Linux

## 🏗 Project Overview

I built an end-to-end CI/CD pipeline that automatically builds, tests, and deploys a Java application to Kubernetes. The pipeline pulls from this Git repository, builds the application using Maven, containerizes it with Docker, pushes to AWS ECR, and deploys to an EKS cluster.

## Pipeline Flow

1. Developer pushes code to Git repository
2. Jenkins detects the change and triggers the pipeline
3. CI Pipeline:
   - Increments version in pom.xml
   - Builds Java artifact using Maven
   - Creates Docker image
   - Pushes image to private ECR repository
4. CD Pipeline:
   - Pulls latest image from ECR
   - Updates Kubernetes manifests
   - Deploys to EKS cluster
   - Commits version changes back to repository

## 📋 Implementation Steps

1. Created a private repository in AWS ECR to store the Docker images

![ecr](https://github.com/Princeton45/eks-ecr-complete-pipeline/blob/main/images/aws-ecr.png)

2. Added the ECR credentials into Jenkins so Jenkins can authenticate

![ecr-creds](https://github.com/Princeton45/eks-ecr-complete-pipeline/blob/main/images/ecr-credentials.png)

3. Set up an EKS cluster in AWS for application deployment

![eks](https://github.com/Princeton45/eks-ecr-complete-pipeline/blob/main/images/eks.png)


4. Configured Jenkins with necessary AWS and Kubernetes credentials

![aws-access](https://github.com/Princeton45/eks-ecr-complete-pipeline/blob/main/images/aws-access.png)

5. Created a docker regsitry secret in EKS so Kubernetes can fecth an image in the AWS ECR repository.

`kubectl create secret docker-registry aws-registry-key --docker-server 251770415749.dkr.ecr.us-east-1.amazonaws.com --docker-username=AWS --docker-password=<password>`

![kube-secret](https://github.com/Princeton45/eks-ecr-complete-pipeline/blob/main/images/kube-secret.png)


6. Created a Jenkinsfile that handles:
   - Version incrementation
   - Maven artifact building
   - Docker image building and pushing to ECR
   - Kubernetes deployment
   - Version update commits

[Suggested Image: Screenshot of successful Jenkins pipeline run]

## 🔄 Pipeline Flow

```plaintext
Git Repository -> Jenkins Pipeline -> Maven Build -> Docker Image -> AWS ECR -> AWS EKS