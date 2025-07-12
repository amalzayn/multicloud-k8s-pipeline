pipeline {
    agent any
    
    parameters {
        choice(
            name: 'CLOUD_PROVIDER',
            choices: ['GKE', 'EKS', 'BOTH'],
            description: 'Select cloud provider for deployment'
        )
        choice(
            name: 'ACTION',
            choices: ['PLAN', 'APPLY', 'DESTROY'],
            description: 'Select Terraform action'
        )
        booleanParam(
            name: 'DEPLOY_APP',
            defaultValue: false,
            description: 'Deploy sample application after infrastructure'
        )
    }
    
    environment {
        LOCAL_REPO_PATH = "/Users/ftzayn/Desktop/multi-cloud1/projectmain"
    }
    
    stages {
        stage('Verify Local Repository') {
            steps {
                script {
                    if (!fileExists("${LOCAL_REPO_PATH}/Jenkinsfile")) {
                        error "Jenkinsfile not found in ${LOCAL_REPO_PATH}"
                    }
                    echo "Local repository verified at ${LOCAL_REPO_PATH}"
                }
            }
        }
        
        stage('Deploy to GKE') {
            when {
                anyOf {
                    params.CLOUD_PROVIDER == 'GKE'
                    params.CLOUD_PROVIDER == 'BOTH'
                }
            }
            steps {
                script {
                    echo "🚀 Working with GKE..."
                    
                    sh """
                        cd ${LOCAL_REPO_PATH}/gke
                        terraform init
                        terraform fmt -check || terraform fmt
                        terraform validate
                        terraform plan -out=tfplan-gke
                    """
                    
                    if (params.ACTION == 'APPLY') {
                        sh """
                            cd ${LOCAL_REPO_PATH}/gke
                            terraform apply -auto-approve tfplan-gke
                        """
                    } else if (params.ACTION == 'DESTROY') {
                        sh """
                            cd ${LOCAL_REPO_PATH}/gke
                            terraform destroy -auto-approve
                        """
                    }
                }
            }
        }
        
        stage('Deploy to EKS') {
            when {
                anyOf {
                    params.CLOUD_PROVIDER == 'EKS'
                    params.CLOUD_PROVIDER == 'BOTH'
                }
            }
            steps {
                script {
                    echo "🚀 Working with EKS..."
                    
                    sh """
                        cd ${LOCAL_REPO_PATH}/eks
                        terraform init
                        terraform fmt -check || terraform fmt
                        terraform validate
                        terraform plan -out=tfplan-eks
                    """
                    
                    if (params.ACTION == 'APPLY') {
                        sh """
                            cd ${LOCAL_REPO_PATH}/eks
                            terraform apply -auto-approve tfplan-eks
                        """
                    } else if (params.ACTION == 'DESTROY') {
                        sh """
                            cd ${LOCAL_REPO_PATH}/eks
                            terraform destroy -auto-approve
                        """
                    }
                }
            }
        }
        
        stage('Deploy Applications') {
            when {
                allOf {
                    params.DEPLOY_APP == true
                    params.ACTION == 'APPLY'
                }
            }
            steps {
                script {
                    if (params.CLOUD_PROVIDER == 'GKE' || params.CLOUD_PROVIDER == 'BOTH') {
                        echo "📦 Deploying app to GKE..."
                        sh """
                            gcloud container clusters get-credentials my-cluster --region us-central1
                            kubectl apply -f ${LOCAL_REPO_PATH}/apps/sample-app/
                            kubectl get pods
                            kubectl get services
                        """
                    }
                    
                    if (params.CLOUD_PROVIDER == 'EKS' || params.CLOUD_PROVIDER == 'BOTH') {
                        echo "📦 Deploying app to EKS..."
                        sh """
                            cd ${LOCAL_REPO_PATH}/eks
                            CLUSTER_NAME=\$(terraform output -raw cluster_name)
                            aws eks update-kubeconfig --name \$CLUSTER_NAME --region us-east-2
                            kubectl apply -f ${LOCAL_REPO_PATH}/apps/sample-app/
                            kubectl get pods
                            kubectl get services
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "✅ Operation completed successfully!"
        }
        failure {
            echo "❌ Operation failed!"
        }
    }
}