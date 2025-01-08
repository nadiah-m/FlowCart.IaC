/* groovylint-disable NestedBlockDepth */
pipeline {
    agent any
    tools {
        terraform 'Terraform'
    }
    environment {
        TF_VAR_region = 'us-east-1' // AWS region
        TF_DIR = './terraform' // Directory where your Terraform code resides
        K8_DIR = './Kubernetes'
        EKS_CLUSTER = 'flowcart-eks'
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

        stage('Get Kube config file') {
            steps {
                //withAWS(credentials:'AWSCredentials', region: 'us-east-1') {
                    script {
                        dir("${K8_DIR}") {
                            sh "aws eks update-kubeconfig --region ${TF_VAR_region} --name ${EKS_CLUSTER}"
                            /* groovylint-disable-next-line LineLength */
                            //sh 'kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml'
                            // sh 'kubectl apply -f ingress.yaml'
                            // sh 'kubectl apply -f frontend-service.yaml'
                            // sh 'kubectl apply -f frontend-dep.yaml'
                        }
                    }
                //}
            }
        }

        stage('Install Ingress controller') {
            steps {
                //withAWS(credentials:'AWSCredentials', region: 'us-east-1') {
                    script {
                        dir("${K8_DIR}") {
                            /* groovylint-disable-next-line LineLength */
                            sh 'kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml'
                            // sh 'kubectl apply -f ingress.yaml'
                            // sh 'kubectl apply -f frontend-service.yaml'
                            // sh 'kubectl apply -f frontend-dep.yaml'
                        }
                    }
                //}
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
