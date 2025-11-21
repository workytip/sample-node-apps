pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/workytip/sample-node-apps.git'
            }
        }
        
        stage('Setup Build Tools') {
            steps {
                script {
                    sh '''
                    # Install Docker, kubectl, etc.
                    curl -fsSL https://get.docker.com -o get-docker.sh
                    sh get-docker.sh
                    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                    chmod +x kubectl
                    '''
                }
            }
        }
        
        stage('Build Docker Images') {
            steps {
                script {
                    sh '''
                    docker build -t workytip/node-app1:latest -f app1/Dockerfile app1/
                    docker build -t workytip/node-app2:latest -f app2/Dockerfile app2/
                    '''
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push workytip/node-app1:latest
                    docker push workytip/node-app2:latest
                    '''
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    git clone https://github.com/workytip/k8s-applications.git
                    ./kubectl apply -f k8s-applications/2-applications/
                    ./kubectl rollout restart deployment/app1 -n app
                    ./kubectl rollout restart deployment/app2 -n app
                    '''
                }
            }
        }
    }
}