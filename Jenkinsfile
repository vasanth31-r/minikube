pipeline {
    agent any

    tools {
        sonarScanner 'SonarScanner'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/vasanth31-r/minikube.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarScanner') { // Ensure this name matches your Jenkins configuration
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=minikube -Dsonar.sources=."
                    }
                }
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Use the Minikube Docker daemon
                    sh 'eval $(minikube docker-env)'
                    
                    // Build the Docker image
                    def appImage = docker.build("vasanth31r/minikube-app:${env.BUILD_NUMBER}", "./k8s")
                    
                    // Push the image to the Minikube's internal Docker registry
                    appImage.push()
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    // Update the deployment.yaml with the new image tag
                    sh "sed -i 's|vasanth31r/minikube-app:.*|vasanth31r/minikube-app:${env.BUILD_NUMBER}|' k8s/deployment.yaml"
                    
                    // Apply the deployment to Minikube
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        }
    }
}
