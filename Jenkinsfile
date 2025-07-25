pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'zainab7861/zainab-sparta-app'
        TARGET_VM = '108.130.50.28'
    }

    stages {
        stage('Checkout dev') {
            steps {
                git branch: 'dev',
                    url: 'git@github.com:zainabx78/tech501-sparta-app-cicd.git',
                    credentialsId: 'github-ssh-key'
            }
        }

        stage('Merge dev into main') {
            steps {
                sshagent(['github-ssh-key']) {
                    sh '''
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins"
                        git checkout main
                        git pull origin main --rebase
                        git merge dev --no-ff -m "merged dev to main with jenkins"
                        git remote set-url origin git@github.com:zainabx78/tech501-sparta-app-cicd.git
                        git push origin main

                        git checkout dev
                        git merge main --no-ff -m "synced dev with main"
                        git push origin dev
                    '''
                }
            }
        }

        stage('Build Docker image') {
            steps {
                script {
                    env.IMAGE_TAG = "${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    dockerImage = docker.build(env.IMAGE_TAG, 'app')
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-creds') {
                            dockerImage.push()
                        }
                    }
                }
            }
        }

        stage('Deploy to Minikube.') {
            steps {
                script {
                    def dockerImage = env.IMAGE_TAG

                    // Copy the K8s manifest file to the remote server
                    sh "scp -i /var/lib/jenkins/.ssh/aws-key-zainab.pem k8s/sparta-app.yml ubuntu@${TARGET_VM}:/tmp/"

                    // SSH into the remote server and apply the manifest & update the deployment image
                sh """#!/bin/bash
ssh -i /var/lib/jenkins/.ssh/aws-key-zainab.pem ubuntu@${TARGET_VM} <<EOF
kubectl apply -f /tmp/sparta-app.yml
kubectl set image deployment/nodejs-deployment nodejs-app=${dockerImage} --record
EOF
"""

                }
            }
        }
    }
}
