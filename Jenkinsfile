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
        MASKED_SSH_KEY = '/tmp/ssh-*'
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
                    sh "docker push ${env.ECR_REPO}:${env.TAG}" //for push we use with credentials
                } //withCredentials
            } //steps
//             post {
//                 success {
//                     emailext(
//                         subject: "Jenkins Job - Docker Image Pushed to ECR Successfully",
//                         body: "Hello,\n\nThe Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR.\n\nBest regards,\nJenkins",
//                         recipientProviders: [[$class: 'DevelopersRecipientProvider']],
//                         to: "adityanavaneethan98@gmail.com"
//                     )
//                 } //success
//             } //post
        } //stage
        stage('Static Code Analysis - SonarQube') {
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh 'mvn sonar:sonar -Dsonar.organization=aditya1234556'
                    } //withSonarQubeEnv
                } //script
            } //steps
        } //stage

        stage('Container Security Scan - Trivy') {
            steps {
                script {
                    sh "trivy --timeout 1m image ${ECR_REPO}:${TAG} > 'trivyscan.txt'"
                } //script
            } //steps
//             post {
//                 success{
//                     emailext(
//                         subject: "Trivy scan result",
//                         body: "Hello, \n Trivy scan result in attachment \n Best regards, \n Jenkins",
//                         recipientProviders: [[$class: 'DevelopersRecipientProvider']],
//                         to: "adityanavaneethan98@gmail.com",
//                         attachmentsPattern: 'trivyscan.txt'
//                     )
//                 } //success
//             } //post
        } //stage

        stage('Deploy to Environment test') {
            steps {
                    sshagent(['ec2-ssh-key']) {
                    sh 'echo "Starting SSH connection test"'
                    sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@35.179.95.72 ls'
                } //sshagent
            } //steps
        } //stage

        stage('Deploy to Environment') {
            steps {
                script {
                    def targetHost = ''
                    if (env.BRANCH_NAME == 'DEV') {
                        targetHost = '13.42.35.221'
                    } else if (env.BRANCH_NAME == 'STAGING') {
                        targetHost = '35.179.105.161'
                    } else if (env.BRANCH_NAME == 'PROD') {
                        targetHost = '35.179.95.72'
                    } else if (env.BRANCH_NAME == 'master') {
                        targetHost = '35.179.95.72'
                    }
                    withCredentials([usernamePassword(credentialsId: 'aws-ecr', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sshagent(['ec2-ssh-key']){
                    sh """
                    ssh -tt -o StrictHostKeyChecking=no ubuntu@${targetHost} << EOF
                    aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin ${env.ECR_REPO}
                    docker pull ${ECR_REPO}:${TAG}
                    docker stop ${IMAGE_NAME} || true
                    docker rm ${IMAGE_NAME} || true
                    docker run -d --name ${IMAGE_NAME} -p 80:80 ${ECR_REPO}:${TAG}
                    EOF
                    """
                    } //withCredentials
                    } //sshagent
                } //script
            } //steps
        } //stage (Deploy to Environment)
    } //stages
} //pipeline

