pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }
    tools {
        maven "mvn"
    }

    environment {
        IMAGE_NAME = "chayma0722/crud-spring"
        TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/chaimaarbaoui-ux/crud-spring'
            }
        }
        stage("Tests") {
            steps {
                sh "mvn clean test"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-26.2.0.119303') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${TAG} .
                docker tag ${IMAGE_NAME}:${TAG} ${IMAGE_NAME}:latest
                """
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-cred',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}:${TAG}
                    docker push ${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage("Deploy") {
            steps {
                withCredentials([sshUserPrivateKey(
                        credentialsId: 'vm-ssh-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                )]) {
                    sh """
            chmod 600 $SSH_KEY
            ssh -o StrictHostKeyChecking=no -i $SSH_KEY $SSH_USER@74.234.49.53 << 'EOF'
              cd /home/chayma/crud-app
              sed -i "s/BACKEND_TAG=.*/BACKEND_TAG=${TAG}/" .env
              docker-compose pull backend
              docker-compose up -d backend
            EOF
            """
                }
            }
        }


    }
}
