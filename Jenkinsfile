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
    stages{
        stage('Test SSH_KEY'){
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh 'echo "SSH Key loaded successfully"'
                }
            }
        }
    }
}