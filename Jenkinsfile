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
        
        stage('Build with Kaniko') {
            steps {
                script {
                    // Kaniko will build without Docker daemon
                    sh '''
                    # Create Docker config for Kaniko
                    mkdir -p /kaniko/.docker
                    echo "{\\"auths\\":{\\"https://index.docker.io/v1/\\":{\\"auth\\":\\"$(echo -n $DOCKERHUB_CREDENTIALS_USR:$DOCKERHUB_CREDENTIALS_PSW | base64)\\"}}}" > /kaniko/.docker/config.json
                    
                    # Build images with Kaniko
                    /kaniko/executor --context=app1 --destination=workytip/node-app1:latest
                    /kaniko/executor --context=app2 --destination=workytip/node-app2:latest
                    '''
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    kubectl apply -f k8s-applications/2-applications/
                    kubectl rollout restart deployment/app1 -n applications
                    kubectl rollout restart deployment/app2 -n applications
                    '''
                }
            }
        }
    }
}