pipeline {
    agent any

    tools {
        // Use the correct, full tool type as per the error message.
        // The name "SonarScannertool" must match the name you configured in Jenkins.
        hudson.plugins.sonar.SonarRunnerInstallation 'SonarScannertool'
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
                    // This now correctly points to the tool you've named 'SonarScannertool'
                    def scannerHome = tool 'SonarScannertool'
                    
                    // The 'withSonarQubeEnv' should use the name you gave to the SonarQube Server itself,
                    // not the tool. Check Manage Jenkins -> Configure System for the correct name.
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
