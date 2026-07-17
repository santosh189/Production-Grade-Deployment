pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "harisantosh/production-grade-deployment"
        GIT_USER  = "santosh189"
        GIT_EMAIL = "santosh310santosh@gmail.com"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Docker Image') {
            when {
                branch 'main'
            }

            steps {
                script {
                    def IMAGE_TAG = "build-${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = IMAGE_TAG

                    withCredentials([usernamePassword(
                        credentialsId: 'DOCKER-CREDS',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        sh """
                            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                            echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                            docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Update Kubernetes Manifest') {
            when {
                branch 'main'
            }

            steps {
                script {

                    withCredentials([usernamePassword(
                        credentialsId: 'GITHUB-CREDS',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {

                        sh """
                            git config user.name "${GIT_USER}"
                            git config user.email "${GIT_EMAIL}"

                            git checkout main
                            git pull origin main

                            sed -i "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" k8s/deployment.yml

                            git add k8s/deployment.yml

                            git diff --cached --quiet || git commit -m "Update image to ${IMAGE_TAG}"

                            git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/santosh189/Production-Grade-Deployment.git main
                        """
                    }
                }
            }
        }
    }
}

