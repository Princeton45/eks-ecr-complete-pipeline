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
        }  // Added this closing brace

        stage('commit version update'){
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'jenkins-github', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh "git config --global user.email 'jenkins@example.com'"
                        sh "git config --global user.name 'Jenkins'"
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/Princeton45/eks-ecr-complete-pipeline.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:master'
                    }
                }
            }
        }
    }
}