pipeline {
    agent any

    stages {
        stage('Clean workspace') {
            steps {
                deleteDir()
            }
        }
        stage('Clean Docker images') {
            steps {
                sh 'docker rmi ashnike/healthcare:1.0 || true' // Remove the specific image tag
                sh 'docker rmi ashnike/healthcare:latest || true' // Remove the specific image tag
                sh 'docker image prune -f' // Remove dangling images
            }
        }

        stage('git checkout') {
            steps {
                echo 'Checking out the code from GitHub'
                git 'https://github.com/ashnike/healthcare.git'
            }
        }
        stage('junit stage test'){
            steps {
            echo 'Running Junit stages'
            sh 'mvn test'
           }
        }
        stage('maven build') {
            steps {
                sh 'mvn clean package'
            }
        }
       
        stage('build docker image') {
            steps {
                sh 'docker build -t ashnike/healthcare:1.0 .'
            }
        }

        stage('push docker image to docker hub registry') {
            steps {
                echo 'Pushing image to the registry'
                withCredentials([string(credentialsId: 'dockerpass', variable: 'dockerHubPassword')]) {
                    sh "docker login -u ashnike -p ${dockerHubPassword}"
                    sh 'docker push ashnike/healthcare:1.0'
                }
            }
        }
         stage('Create EKS  Test Cluster using Terraform') {
            steps {
                dir('/home/ubuntu/testcluster') {
                    script {
                        sh """
                            export TF_VAR_aws_access_key=""
                            export TF_VAR_aws_secret_key=""
                            terraform init
                            terraform plan
                            terraform apply -auto-approve
                        """
                    }
                }
            }
        }
        stage('Connect to EKS') {
            steps {
                script {
                    sh """
                    aws configure set aws_access_key_id "AKIAXVN2IFKGFQMLI5FO"
                    aws configure set aws_secret_access_key "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"
                    export AWS_DEFAULT_REGION=ap-south-1
                    aws eks update-kubeconfig --name test-server --region ap-south-1
                    """
                }
            }
        }

        stage('Deployment on test-EKS Cluster') {
            steps {
                script {
                    sh """
                    kubectl apply -f deployment.yml
                    """
                }
            }
        }

        stage('Run the Selenium runnable jar') {
            steps {
                sh 'java -jar Medicure.jar'
            }
        }
         stage('publishrep') {
            steps {
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, includes: 'screenshot.png', keepAll: false, reportDir: '/var/lib/jenkins/workspace/HealthcareApp', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
            }
        }
        stage('Create EKS  Prod Cluster using Terraform') {
            steps {
                dir('/home/ubuntu/prodcluster') {
                    script {
                        sh """
                            export TF_VAR_aws_access_key=""
                            export TF_VAR_aws_secret_key=""
                            terraform init
                            terraform plan
                            terraform apply -auto-approve
                        """
                    }
                }
            }
        }
         stage('Deployment on Prod-EKS Cluster') {
            steps {
                script {
                    sh """
                    kubectl apply -f prod-deployment.yml
                    """
                }
            }
        }
    }
}
