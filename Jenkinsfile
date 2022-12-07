#!/usr/bin/env groovy

pipeline {
    agent any
    tools {
        maven 'maven-3.8.6'
    }
    environment{
        DOCKER_REPO_SERVER = "985079440022.dkr.ecr.us-east-1.amazonaws.com"
        DOCKER_REPO = '985079440022.dkr.ecr.us-east-1.amazonaws.com/my-app'
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
                    echo "building the application..."
                    sh 'mvn clean package'
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'ecr-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                        sh "echo $PASS | docker login -u $USER --password-stdin ${DOCKER_REPO_SERVER}"
                        sh "docker push ${DOCKER_REPO}:${IMAGE_NAME} ${DOCKER_REPO_SERVER}"
                    }
                }
            }
        }
        stage('deploy') {
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
                APP_NAME = 'java-maven-app'
            }
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    withCredentials([usernamePassword(credentialsId: 'github-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "kubectl create secret docker-registry my-registry-key --docker-server=docker.io --docker-username=$USER --docker-password=$PASS"
                        sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'
                        sh 'envsubst < kubernetes/services.yaml | kubectl apply -f -'
                    }
                }
            }
        }
            stage('commit version update') {
                steps {
                    script {
                        withCredentials([usernamePassword(credentialsId: 'gitlab-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                            // git config here for the first time run
                            sh 'git config --global user.email "jenkins@example.com"'
                            sh 'git config --global user.name "jenkins"'

                            sh "git remote set-url origin https://${USER}:${PASS}@gitlab.com/marv254/java-maven-app.git"
                            sh 'git add .'
                            sh 'git commit -m "ci: version bump"'
                            sh 'git push origin HEAD:jenkins-jobs'
                        }
                    }
                }
            }
        }
    }
