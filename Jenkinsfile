pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/workytip/sample-node-apps.git'
            }
        }
        
        stage('Build Docker Images') {
            steps {
                script {
                    sh '''
                    # Install Docker in Jenkins pod
                    curl -fsSL https://get.docker.com -o get-docker.sh
                    sh get-docker.sh
                    sudo usermod -aG docker jenkins
                    
                    # Build images
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
                    # Jenkins has cluster access via service account
                    kubectl apply -f k8s-applications/2-applications/
                    kubectl rollout restart deployment/app1 -n app
                    kubectl rollout restart deployment/app2 -n app
                    '''
                }
            }
        }
    }
}