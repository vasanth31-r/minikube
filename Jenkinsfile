pipeline {
    agent any

    environment {
        PROJECT_IMAGE = "react-app"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Install & Test') {
            steps {
                sh 'npm ci'
                sh 'npm run build --if-present'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                      sonar-scanner \
                        -Dsonar.projectKey=react-app \
                        -Dsonar.sources=src \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    """
                }
            }
        }

        stage('Build Docker Image (Minikube)') {
            steps {
                sh '''
                  eval $(minikube -p minikube docker-env)
                  docker build -t ${PROJECT_IMAGE}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                  if kubectl get deployment react-app >/dev/null 2>&1; then
                    kubectl set image deployment/react-app react-app=${PROJECT_IMAGE}:${BUILD_NUMBER} --record
                  else
                    kubectl apply -f k8s/deployment.yaml
                    kubectl set image deployment/react-app react-app=${PROJECT_IMAGE}:${BUILD_NUMBER} --record
                  fi
                  kubectl rollout status deployment/react-app --timeout=120s
                '''
            }
        }
    }

    post {
        success { echo "✅ Deployment successful" }
        failure { echo "❌ Pipeline failed" }
    }
}
