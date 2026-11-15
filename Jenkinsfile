
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
                sh 'docker build --target final -t voting-app-vote:${BUILD_NUMBER} ./vote'
            }
        }

        stage('Automated Integration Test') {
            steps {
                echo 'Running automated integration tests'
                sh 'docker compose -p voting-test-${BUILD_NUMBER} -f result/docker-compose.test.yml up --build --exit-code-from sut'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up test containers and volumes'
            sh 'docker compose -p voting-test-${BUILD_NUMBER} -f result/docker-compose.test.yml down -v'
        }
    }
}

