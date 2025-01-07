pipeline {
    agent any
    tools {
        terraform 'Terraform'
    }
    environment {
        // TF_VAR_aws_access_key_id = credentials('aws-access-key-id') // Jenkins credentials for AWS access
        // TF_VAR_aws_secret_access_key = credentials('aws-secret-access-key') // Jenkins credentials for AWS secret key
        TF_VAR_region = 'us-east-1' // AWS region
        TF_DIR = './terraform' // Directory where your Terraform code resides
        K8_DIR = './Kubernetes'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout your repository with Terraform code
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                script {
                    // Initialize Terraform (downloads plugins and sets up backend)
                    dir("${TF_DIR}") {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                script {
                    // Validate Terraform configuration
                    dir("${TF_DIR}") {
                        sh 'terraform validate'
                    }
                }
            }
        }

        stage('Terraform format') {
            steps {
                script {
                    dir("${TF_DIR}") {
                        sh 'terraform fmt'
                    }
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                script {
                    // Create Terraform plan and save it to a file
                    dir("${TF_DIR}") {
                        sh 'terraform plan -out=tfplan'
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                script {
                    // Apply the Terraform plan
                    dir("${TF_DIR}") {
                        // Automatically approve the plan to apply changes
                        sh 'terraform apply -auto-approve tfplan'
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    dir("${K8_DIR}") {
                        sh 'aws eks update-kubeconfig --name flowcart-eks'
                        sh 'kubectl apply -f ./'
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    // Clean up Terraform plan file
                    dir("${TF_DIR}") {
                        sh 'rm -f tfplan'
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Terraform apply was successful!'
        }
        failure {
            echo 'Terraform apply failed.'
        }
        always {
            // Optionally, add any clean-up or post-processing here
            echo 'Pipeline execution completed.'
        }
     }
}
