pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command: ["/busybox/sh", "-c"]
    args: ["sleep 9999999"]
    volumeMounts:
    - name: workspace
      mountPath: /workspace
  volumes:
  - name: workspace
    emptyDir: {}
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
        
        stage('Build with Kaniko') {
            steps {
                container('kaniko') {
                    script {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh '''
                            mkdir -p /kaniko/.docker
                            echo "{\\"auths\\":{\\"https://index.docker.io/v1/\\":{\\"auth\\":\\"$(echo -n $DOCKER_USER:$DOCKER_PASS | base64 | tr -d '\\n')\\"}}}" > /kaniko/.docker/config.json
                            
                            # Build from Jenkins workspace
                            /kaniko/executor --context=${WORKSPACE}/app1 --destination=workytip/node-app1:latest
                            /kaniko/executor --context=${WORKSPACE}/app2 --destination=workytip/node-app2:latest
                            '''
                        }
                    }
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