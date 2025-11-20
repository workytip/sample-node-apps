pipeline {
    agent {
        kubernetes {
            defaultContainer 'docker'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:latest
    command: ['cat']
    tty: true
    volumeMounts:
    - mountPath: "/var/run/docker.sock"
      name: "docker-sock"
  volumes:
  - name: docker-sock
    hostPath:
      path: "/var/run/docker.sock"
'''
        }
    }
    
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
                container('docker') {
                    script {
                        sh '''
                        docker build -t workytip/node-app1:latest -f app1/Dockerfile app1/
                        docker build -t workytip/node-app2:latest -f app2/Dockerfile app2/
                        '''
                    }
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                container('docker') {
                    script {
                        sh '''
                        echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                        docker push workytip/node-app1:latest
                        docker push workytip/node-app2:latest
                        '''
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                container('docker') {
                    script {
                        sh '''
                        # Install kubectl in the docker container
                        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x kubectl
                        ./kubectl apply -f k8s/
                        ./kubectl rollout restart deployment/app1 -n app
                        ./kubectl rollout restart deployment/app2 -n app
                        '''
                    }
                }
            }
        }
    }
}