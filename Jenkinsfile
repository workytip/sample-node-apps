pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yourusername/sample-node-apps.git'
            }
        }
        
        stage('Build Docker Images') {
            steps {
                script {
                    sh '''
                    docker build -t your-dockerhub-username/node-app1:latest -f Dockerfile.app1 .
                    docker build -t your-dockerhub-username/node-app2:latest -f Dockerfile.app2 .
                    '''
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push your-dockerhub-username/node-app1:latest
                    docker push your-dockerhub-username/node-app2:latest
                    '''
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    kubectl apply -f k8s-applications/2-applications/
                    kubectl rollout restart deployment/app1 -n app
                    kubectl rollout restart deployment/app2 -n app
                    '''
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
