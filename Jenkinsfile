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
        TERRAFORM_PATH = "/opt/homebrew/bin/terraform"
        GCLOUD_PATH = "/opt/homebrew/bin/gcloud"
        PATH = "/opt/homebrew/bin:${env.PATH}"
    }
    
    stages {
        stage('Verify Local Repository') {
            steps {
                script {
                    if (!fileExists("${LOCAL_REPO_PATH}/Jenkinsfile")) {
                        error "Jenkinsfile not found in ${LOCAL_REPO_PATH}"
                    }
                    echo "Local repository verified at ${LOCAL_REPO_PATH}"
                    
                    // Check if tools are available
                    sh """
                        echo "Checking tools..."
                        which terraform || echo "Terraform not found in PATH"
                        ${TERRAFORM_PATH} version || echo "Terraform not found at ${TERRAFORM_PATH}"
                        which gcloud || echo "gcloud not found in PATH"
                    """
                }
            }
        }
        
        stage('Deploy to GKE') {
            when {
                anyOf {
                    expression { params.CLOUD_PROVIDER == 'GKE' }
                    expression { params.CLOUD_PROVIDER == 'BOTH' }
                }
            }
            steps {
                script {
                    echo "🚀 Working with GKE..."
                    
                    sh """
                        cd ${LOCAL_REPO_PATH}/gke
                        ${TERRAFORM_PATH} init
                        ${TERRAFORM_PATH} fmt -check || ${TERRAFORM_PATH} fmt
                        ${TERRAFORM_PATH} validate
                        ${TERRAFORM_PATH} plan -out=tfplan-gke
                    """
                    
                    if (params.ACTION == 'APPLY') {
                        sh """
                            cd ${LOCAL_REPO_PATH}/gke
                            ${TERRAFORM_PATH} apply -auto-approve tfplan-gke
                        """
                    } else if (params.ACTION == 'DESTROY') {
                        sh """
                            cd ${LOCAL_REPO_PATH}/gke
                            ${TERRAFORM_PATH} destroy -auto-approve
                        """
                    }
                }
            }
        }
        
        stage('Deploy to EKS') {
            when {
                anyOf {
                    expression { params.CLOUD_PROVIDER == 'EKS' }
                    expression { params.CLOUD_PROVIDER == 'BOTH' }
                }
            }
            steps {
                script {
                    echo "🚀 Working with EKS..."
                    
                    sh """
                        cd ${LOCAL_REPO_PATH}/eks
                        ${TERRAFORM_PATH} init
                        ${TERRAFORM_PATH} fmt -check || ${TERRAFORM_PATH} fmt
                        ${TERRAFORM_PATH} validate
                        ${TERRAFORM_PATH} plan -out=tfplan-eks
                    """
                    
                    if (params.ACTION == 'APPLY') {
                        sh """
                            cd ${LOCAL_REPO_PATH}/eks
                            ${TERRAFORM_PATH} apply -auto-approve tfplan-eks
                        """
                    } else if (params.ACTION == 'DESTROY') {
                        sh """
                            cd ${LOCAL_REPO_PATH}/eks
                            ${TERRAFORM_PATH} destroy -auto-approve
                        """
                    }
                }
            }
        }
        
        stage('Deploy Applications') {
            when {
                allOf {
                    expression { params.DEPLOY_APP == true }
                    expression { params.ACTION == 'APPLY' }
                }
            }
            steps {
                script {
                    if (params.CLOUD_PROVIDER == 'GKE' || params.CLOUD_PROVIDER == 'BOTH') {
                        echo "📦 Deploying app to GKE..."
                        sh """
                            ${GCLOUD_PATH} container clusters get-credentials my-cluster --region us-central1
                            kubectl apply -f ${LOCAL_REPO_PATH}/apps/sample-app/
                            kubectl get pods
                            kubectl get services
                        """
                    }
                    
                    if (params.CLOUD_PROVIDER == 'EKS' || params.CLOUD_PROVIDER == 'BOTH') {
                        echo "📦 Deploying app to EKS..."
                        sh """
                            cd ${LOCAL_REPO_PATH}/eks
                            CLUSTER_NAME=\$(${TERRAFORM_PATH} output -raw cluster_name)
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