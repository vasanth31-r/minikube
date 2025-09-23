pipeline {
    agent any

    tools {
        // The correct syntax is to use the tool type as the keyword.
        // It should be 'sonarScanner' as originally intended, but we need to check Jenkins' configuration.
        // A direct fix is to remove the tools block and use the `tool` step directly.
        // However, if you want to use the tools block, the correct syntax is:
        sonarScanner 'SonarScannertool'
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
