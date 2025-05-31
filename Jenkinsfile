pipeline {
    agent any

    environment {
        DOCKER_IMAGE_BACKEND = 'nottiey/backend'
        DOCKER_IMAGE_FRONTEND = 'nottiey/frontend'
        BACKEND_PATH = 'backend'
        FRONTEND_PATH = 'frontend'
        GITHUB_REPO = 'https://github.com/nottie-noe/GitOps-React-Java-App.git'
    }

    tools {
        maven 'Maven'      // Jenkins > Global Tool Configuration
        nodejs 'NodeJS'    // Requires NodeJS Plugin
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${GITHUB_REPO}"
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
                    // Build only if script is defined and files exist
                    sh '''
                        if [ -f package.json ] && grep -q '"build":' package.json && [ -f src/index.js ]; then
                            npm run build
                        else
                            echo "Skipping build: build script or index.js not found"
                        fi
                    '''
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
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
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
                withCredentials([usernamePassword(credentialsId: 'github-push-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh '''
                        git config user.name "ci-bot"
                        git config user.email "ci@example.com"

                        sed -i "s|repository:.*|repository: nottiey/backend|" helm/umbrella-chart/charts/backend/values.yaml
                        sed -i "s|repository:.*|repository: nottiey/frontend|" helm/umbrella-chart/charts/frontend/values.yaml

                        git add helm/umbrella-chart/charts/*/values.yaml
                        git commit -m "Update image repos from Jenkins" || echo "No changes to commit"
                        git push https://$GIT_USER:$GIT_PASS@github.com/nottie-noe/GitOps-React-Java-App.git main
                    '''
                }
            }
        }
    }
}
