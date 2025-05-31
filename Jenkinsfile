pipeline {
    agent any

    environment {
        DOCKER_IMAGE_BACKEND = 'nottiey/backend'
        DOCKER_IMAGE_FRONTEND = 'nottiey/frontend'
        BACKEND_PATH = 'backend'
        FRONTEND_PATH = 'frontend'
    }

    tools {
        maven 'Maven'      // Must be configured in Jenkins > Global Tool Config
        nodejs 'NodeJS'    // Must be configured via NodeJS Plugin
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/nottie-noe/GitOps-React-Java-App.git'
            }
        }

        stage('Build Backend with Maven') {
            steps {
                dir("${BACKEND_PATH}") {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Build Frontend with npm') {
            steps {
                dir("${FRONTEND_PATH}") {
                    sh 'npm install'
                    // fallback if "build" script is not defined
                    sh 'npm run build || echo "Skipping build: no build script defined"'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE_BACKEND}:latest ${BACKEND_PATH}"
                    sh "docker build -t ${DOCKER_IMAGE_FRONTEND}:latest ${FRONTEND_PATH}"
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push ${DOCKER_IMAGE_BACKEND}:latest
                        docker push ${DOCKER_IMAGE_FRONTEND}:latest
                    '''
                }
            }
        }

        stage('Update Helm Values (Optional)') {
            steps {
                script {
                    sh '''
                        git config user.name "ci-bot"
                        git config user.email "ci@example.com"

                        sed -i "s|repository:.*|repository: ${DOCKER_IMAGE_BACKEND}|" helm/umbrella-chart/charts/backend/values.yaml
                        sed -i "s|repository:.*|repository: ${DOCKER_IMAGE_FRONTEND}|" helm/umbrella-chart/charts/frontend/values.yaml

                        git add helm/umbrella-chart/charts/*/values.yaml
                        git commit -m "Update image repos from Jenkins"
                        git push origin main
                    '''
                }
            }
        }
    }
}
