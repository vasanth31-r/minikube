pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/vasanth31-r/minikube.git'
            }
        }
        
        stage('Build') {
            steps {
                // Your build steps (e.g., npm install, npm build)
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('My Local SonarQube') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=react-app \
                    -Dsonar.projectName="React App" \
                    -Dsonar.sources=src \
                    -Dsonar.language=js \
                    -Dsonar.sourceEncoding=UTF-8
                    '''
                }
            }
        }
        
        stage('Quality Gate Check') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    sh 'eval $(minikube docker-env)'
                    def appImage = docker.build("vasanth31r/minikube-app:${env.BUILD_NUMBER}", "-f k8s/Dockerfile .")
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
    
    post {
        always {
            echo 'Pipeline completed. Cleaning up...'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
