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

        stage('Push Docker Images') {
            steps {
                echo 'Pushing all application images to Docker Hub'

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        set +x
                        set -eu

                        export DOCKER_CONFIG="$(mktemp -d)"
                        trap 'docker logout >/dev/null 2>&1 || true; rm -rf "$DOCKER_CONFIG"' EXIT

                        printf '%s' "$DOCKERHUB_TOKEN" | docker login \
                            --username "$DOCKERHUB_USERNAME" \
                            --password-stdin

                        docker tag "voting-app-vote:${BUILD_NUMBER}" \
                            "$DOCKERHUB_USERNAME/voting-app:vote-${BUILD_NUMBER}"

                        docker tag "result-app:${BUILD_NUMBER}" \
                            "$DOCKERHUB_USERNAME/voting-app:result-${BUILD_NUMBER}"

                        docker tag "worker-app:${BUILD_NUMBER}" \
                            "$DOCKERHUB_USERNAME/voting-app:worker-${BUILD_NUMBER}"

                        docker push "$DOCKERHUB_USERNAME/voting-app:vote-${BUILD_NUMBER}"

                        docker push "$DOCKERHUB_USERNAME/voting-app:result-${BUILD_NUMBER}"

                        docker push "$DOCKERHUB_USERNAME/voting-app:worker-${BUILD_NUMBER}"
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                echo 'Deploying the tested application version'

                sh '''
                    set -eu

                    docker compose -p example-voting-app \
                        -f docker-compose.deploy.yml pull vote result worker

                    docker compose -p example-voting-app \
                        -f docker-compose.deploy.yml stop vote result worker

                    docker compose -p example-voting-app \
                        -f docker-compose.deploy.yml up -d --no-build --force-recreate vote result worker

                    docker compose -p example-voting-app \
                        -f docker-compose.deploy.yml ps
                '''
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
