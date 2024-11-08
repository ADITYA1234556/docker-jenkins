pipeline {
    agent {
    label 'slave'
    }

    stages {
        stage('Git verify') {
            steps {
                script{
                    sh 'git --version'
                }
            }
        }
    }
}

