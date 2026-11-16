


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

        stage('Build Vote Image') {
            steps {
                echo 'Building Vote application Docker image'
                sh 'docker build --target final -t voting-app-vote:${BUILD_NUMBER} ./vote'
            }
        }

        stage('Build Result Image') {
            steps {
                echo 'Building Result application Docker image'
                sh 'docker build -t result-app:${BUILD_NUMBER} ./result'
            }
        }

        stage('Build Worker Image') {
            steps {
                echo 'Building Worker application Docker image'
                sh 'docker build -t worker-app:${BUILD_NUMBER} ./worker'
            }
        }

        stage('Automated Integration Test') {
            steps {
                echo 'Running automated integration tests'
                sh 'docker compose -p voting-test-${BUILD_NUMBER} -f result/docker-compose.test.yml up --build --exit-code-from sut'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing tested Vote image to Docker Hub'

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        set +x
                        set -eu

                        trap 'docker logout >/dev/null 2>&1 || true' EXIT

                        printf '%s' "$DOCKERHUB_TOKEN" | docker login \
                            --username "$DOCKERHUB_USERNAME" \
                            --password-stdin

                        docker tag "voting-app-vote:${BUILD_NUMBER}" \
                            "$DOCKERHUB_USERNAME/voting-app:${BUILD_NUMBER}"

                        docker push "$DOCKERHUB_USERNAME/voting-app:${BUILD_NUMBER}"
                    '''
                }
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

