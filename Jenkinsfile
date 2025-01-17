pipeline {
    agent any
    
    environment {
        CLUSTER_NAME = 'main-cluster'
        AWS_REGION = 'us-east-1'
        MONITORING_NAMESPACE = 'monitoring'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Configure kubectl') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh '''
                        aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${AWS_REGION}
                        kubectl config current-context
                    '''
                }
            }
        }
        
        stage('Install CRDs') {
            steps {
                sh '''
                    # Install Prometheus Operator CRDs
                    echo "Applying CRDs for Prometheus and Grafana..."
                    kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/bundle.yaml
                '''
            }
        }
        
        stage('Create Namespace') {
            steps {
                sh '''
                    kubectl create namespace ${MONITORING_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                '''
            }
        }
        
        stage('Deploy Monitoring Stack') {
            steps {
                sh '''
                    # Add Helm repositories
                    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
                    helm repo add grafana https://grafana.github.io/helm-charts
                    helm repo update
                    
                    # Install Prometheus with CRDs managed externally
                    helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
                        -f helm/prometheus/values.yaml \
                        -n ${MONITORING_NAMESPACE} \
                        --create-namespace
                    
                    # Install Grafana
                    helm upgrade --install grafana grafana/grafana \
                        -f helm/grafana/values.yaml \
                        -n ${MONITORING_NAMESPACE}
                '''
            }
        }
        
        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Verifying monitoring deployment..."
                    kubectl get pods -n ${MONITORING_NAMESPACE}
                    kubectl get svc -n ${MONITORING_NAMESPACE}
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Monitoring stack deployed successfully!'
        }
        failure {
            echo 'Deployment failed! Check the logs for details.'
        }
    }
}
