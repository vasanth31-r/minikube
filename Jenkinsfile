pipeline {
    agent any

    tools {
        // Use the correct syntax for SonarScanner tool
        hudsonPluginsSonarSonarRunnerInstallation 'SonarScannertool'
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
                    def scannerHome = tool 'SonarScannertool'
                    
                    withSonarQubeEnv('My Local SonarQube') { 
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=minikube -Dsonar.sources=."
                    }
                }
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh 'eval $(minikube docker-env)'
                    def appImage = docker.build("vasanth31r/minikube-app:${env.BUILD_NUMBER}", "./k8s")
                    appImage.push()
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh "sed -i 's|vasanth31r/minikube-app:.*|vasanth31r/minikube-app:${env.BUILD_NUMBER}|' k8s/deployment.yaml"
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        }
    }
}
