pipeline {
    agent any

    tools {
sonar 'SonarScannertool'
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
                    // This finds the tool you configured.
                    def scannerHome = tool 'SonarScannertool'
                    
                    // The withSonarQubeEnv step should use the name of the SonarQube server itself,
                    // not the tool name. Use the name you configured under Manage Jenkins -> Configure System.
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
