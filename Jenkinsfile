
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

        stage('Build Docker Image') {
            steps {
                echo 'Building Vote application Docker image'
                sh 'docker build --target dev -t voting-app-vote:${BUILD_NUMBER} ./vote'
            }
        }
    }
}
