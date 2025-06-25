pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'zainab7861/zainab-sparta-app'
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
                script {
                    sh '''
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins"
                    git checkout main
		
                    git pull origin main --rebase
                    git merge dev --no-ff -m "merged dev to main with jenkins"
                    git push origin main
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

        stage('Deploy to Minikube') {
            steps {
                sh 'scp -i /var/lib/jenkins/.ssh/aws-key-zainab.pem k8s/sparta-app.yml ubuntu@54.217.157.100:/tmp/'
                sh 'ssh -i /var/lib/jenkins/.ssh/aws-key-zainab.pem ubuntu@54.217.157.100 "kubectl apply -f /tmp/sparta-app.yml"'
            }
        }
    }
}
