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
        
        stage('Create Namespace') {
            steps {
                sh '''
                    kubectl create namespace ${MONITORING_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                    kubectl get namespace ${MONITORING_NAMESPACE}
                '''
            }
        }
        
        stage('Deploy Prometheus') {
            steps {
                sh '''
                    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
                    helm repo update
                    helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
                        -f helm/prometheus/values.yaml \
                        -n ${MONITORING_NAMESPACE} \
                        --create-namespace
                '''
            }
        }
        
        stage('Deploy Grafana') {
            steps {
                sh '''
                    helm repo add grafana https://grafana.github.io/helm-charts
                    helm repo update
                    helm upgrade --install grafana grafana/grafana \
                        -f helm/grafana/values.yaml \
                        -n ${MONITORING_NAMESPACE} \
                        --create-namespace
                '''
            }
        }
        
        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get pods -n ${MONITORING_NAMESPACE}
                    kubectl rollout status deployment -n ${MONITORING_NAMESPACE} -l app.kubernetes.io/name=prometheus
                    kubectl rollout status deployment -n ${MONITORING_NAMESPACE} -l app.kubernetes.io/name=grafana
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
