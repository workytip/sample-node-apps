pipeline {
    agent {
        kubernetes {
            defaultContainer 'kaniko'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command: ["/busybox/sleep"]  # ← CHANGE THIS
    args: ["infinity"]           # ← ADD THIS
    tty: true
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  volumes:
  - name: kaniko-secret
    secret:
      secretName: docker-config
      items:
      - key: .dockerconfigjson
        path: config.json
'''
        }
    }
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }
    
    stages {
        stage('Build and Push Docker Images') {
            steps {
                container('kaniko') {
                    script {
                        sh '''
                        /kaniko/executor \
                          --context=app1/ \
                          --dockerfile=app1/Dockerfile \
                          --destination=workytip/node-app1:latest \
                          --cache=true
                        
                        /kaniko/executor \
                          --context=app2/ \
                          --dockerfile=app2/Dockerfile \
                          --destination=workytip/node-app2:latest \
                          --cache=true
                        '''
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                container('kaniko') {
                    script {
                        sh '''
                        # Install kubectl and deploy
                        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                        chmod +x kubectl
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
}