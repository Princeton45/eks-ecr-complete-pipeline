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


6. Created a CI/CD Jenkinsfile that handles:
   - Version incrementation
   - Maven artifact building
   - Docker image building and pushing to ECR
   - Kubernetes deployment
   - Version update commits

Successful pipeline run:

![jenkins-run](https://github.com/Princeton45/eks-ecr-complete-pipeline/blob/main/images/jenkins-run.png)

The Docker image being uploaded to AWS ECR:

![all-ecr](https://github.com/Princeton45/eks-ecr-complete-pipeline/blob/main/images/all-ecr.png)

![ecr2](https://github.com/Princeton45/eks-ecr-complete-pipeline/blob/main/images/ecr2.png)

AWS EKS (Kubernetes) Deployments:

![K8S](https://github.com/Princeton45/eks-ecr-complete-pipeline/blob/main/images/k8s-deployments.png)



`Jenkinsfile`
```groovy
#!/usr/bin/env groovy

pipeline {
    agent any
    tools {
        maven 'maven-3.9.9'
    }
    environment {
        DOCKER_REPO_SERVER = "251770415749.dkr.ecr.us-east-1.amazonaws.com"
        DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('build app') {
            steps {
                script {
                    echo 'building the application...'
                    sh 'mvn clean package'
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: '08157819-e971-4cdd-b0c0-d6714087f3bd', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                        sh 'echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}'
                        sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('deploy') {
            environment {
                   AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                   AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
                   APP_NAME = 'java-maven-app'
            }
            steps {
                script {
                    echo "deploying docker image..."
                    sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'
                    sh 'envsubst < kubernetes/service.yaml | kubectl apply -f -'
                }
            }
        }

        stage('commit version update'){
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'jenkins-github', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh "git config --global user.email 'jenkins@example.com'"
                        sh "git config --global user.name 'Jenkins'"
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/Princeton45/eks-ecr-complete-pipeline.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:main'
                    }
                }
            }
        }
    }
}
```

