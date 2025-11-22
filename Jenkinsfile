script {
                sh '''
                cd /home/jenkins/agent/workspace/node-apps-pipeline
                git clone https://github.com/workytip/k8s-applications.git
                
                # Apply Kubernetes manifests - this creates the 'app' namespace
                kubectl apply -f k8s-applications/2-applications/
                
                # Restart deployments in the 'app' namespace (not 'applications')
                kubectl rollout restart deployment/app1 -n app
                kubectl rollout restart deployment/app2 -n app
                
                # Wait for rollout to complete
                kubectl rollout status deployment/app1 -n app --timeout=300s
                kubectl rollout status deployment/app2 -n app --timeout=300s
                
                # Verify deployment
                echo "=== Deployment Status in app namespace ==="
                kubectl get deployments -n app
                echo "=== Pod Status in app namespace ==="
                kubectl get pods -n app
                echo "=== Services in app namespace ==="
                kubectl get services -n app
                '''
            }