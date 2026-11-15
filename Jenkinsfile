
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Starting the voting application pipeline'
                checkout scm
            }
        }

        stage('Build Information') {
            steps {
                echo 'Source code downloaded successfully'
                sh 'git --version'
                sh 'docker --version'
            }
        }
    }
}
