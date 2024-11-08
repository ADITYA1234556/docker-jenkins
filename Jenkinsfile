pipeline {
    agent {
    label 'slave'
    }

    environment {
        ECR_REPO = '866934333672.dkr.ecr.eu-west-2.amazonaws.com/adityaimages'
        IMAGE_NAME = 'app-image'
        TAG = "${env.BRANCH_NAME}-${env.BUILD_ID}"
        AWS_REGION = "eu-west-2"
        SSH_KEY = credentials('ec2-ssh-key')
    }

    stages {
        stage('Git verify') {
            steps {
                script{
                    sh 'git --version'
                }
            }
        }
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/ADITYA1234556/docker-jenkins.git', credentialsId: 'github-token'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.ECR_REPO}:${env.TAG}")
                }
            }
        }
        stage('Push to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-ecr', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')])
                {
                    sh "aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin ${env.ECR_REPO}"
                    sh "docker push ${env.ECR_REPO}:${env.TAG}"
                }
            }
                post {
                success {
                    // Send email notification after successful image push to ECR
                    emailext(
                        subject: "Jenkins Job - Docker Image Pushed to ECR Successfully",
                        body: "Hello,\n\nThe Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR.\n\nBest regards,\nJenkins",
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                        to: "adityanavaneethan98@gmail.com"
                    )
                }
            }
        }
        stage('Static Code Analysis - SonarQube') {
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh 'mvn sonar:sonar -Dsonar.organization=aditya1234556'
                    }
                }
            }
        }
        stage('Container Security Scan - Trivy') {
            steps {
                script {
                    sh "trivy --timeout 1m image ${ECR_REPO}:${TAG} > 'trivyscan.txt'"
                    env.TRIVY_SCAN_RESULT = readFile('trivyscan.txt')
                }
            }
            post {
                success{
                emailext(
                subject: "Trivy scan result",
                body: "Hello, \n Trivy scan result in attachment \n Best regards, \n Jenkins",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: "adityanavaneethan98@gmail.com",
                attachmentsPattern: 'trivyscan.txt'
                )
                }
            }
        }
//         stage('Test SSH_KEY'){
//             steps{
//                 withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY')]) {
//                     sh 'echo "SSH Key loaded successfully"'
//                 }
//             }
//         }
        stage('Deploy to Environment') {
            steps {
                script {
                    def targetHost = ''
                    if (env.BRANCH_NAME == 'DEV') {
                        targetHost = '13.42.35.221'
                    } else if (env.BRANCH_NAME == 'STAGING') {
                        targetHost = '13.42.59.250'
                    } else if (env.BRANCH_NAME == 'PROD') {
                        targetHost = '35.178.153.62'
                    } else if (env.BRANCH_NAME == 'master') {
                        targetHost = '35.178.153.62'
                    }
                    sshagent([${SSH_KEY}]){
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${targetHost} << EOF
                        pwd
                        uptime
                        EOF
                        """
                    }
                }
            }
        }
    }
}