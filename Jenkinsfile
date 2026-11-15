
pipeline {
    agent any

    stages {
        stage('Source Code Verification') {
            steps {
                echo 'Source code downloaded successfully'
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('Docker Daemon Check') {
            steps {
                echo 'Checking Docker Engine connection'
                sh 'docker ps'
            }
        }
    }
}
