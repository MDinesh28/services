pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        EKS_CLUSTER_NAME = "EKS-1"
        KUBERNETES_NAMESPACE = "default"
        GIT_REPO = "https://github.com/MDinesh28/services.git"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    script {
                        // Configure AWS and deploy
                        sh """
                            export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                            export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                            
                            aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER_NAME
                            kubectl apply -f k8s/deployment.yml
                            kubectl apply -f k8s/service.yml
                        """

                        // Wait for LoadBalancer provisioning
                        sleep(time: 10, unit: 'SECONDS')
                        
                        // Get and print service details
                        def SERVICE_DETAILS = sh(
                            script: "kubectl get svc -n ${KUBERNETES_NAMESPACE} -o wide",
                            returnStdout: true
                        ).trim()
                        
                        echo "=============================================="
                        echo "✅ Deployment Successful - Service Details:"
                        echo "${SERVICE_DETAILS}"
                        echo "=============================================="

                        // Extract and print LoadBalancer URL if available
                        def LB_URL = sh(
                            script: "kubectl get svc -n ${KUBERNETES_NAMESPACE} -o jsonpath='{.items[?(@.spec.type==\"LoadBalancer\")].status.loadBalancer.ingress[0].hostname}'",
                            returnStdout: true
                        ).trim()
                        
                        if (LB_URL) {
                            echo "🌐 LoadBalancer URL: http://${LB_URL}"
                        } else {
                            echo "⚠️ No LoadBalancer service found"
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline succeeded! Check above for service details."
        }
        failure {
            echo "❌ Pipeline failed! Check logs for errors."
        }
    }
}
