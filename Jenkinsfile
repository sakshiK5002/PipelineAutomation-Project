pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')
        DOCKER_IMAGE = 'YOUR_DOCKERHUB_USERNAME/sonarproject'
        SONAR_URL = 'http://16.16.194.221:9000'
    }

    stages {

        stage('Checkout from GitHub') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/cjcnkoIhe/sonarproject.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh '''
                    java -version
                    mvn -version
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('SonarQube Code Analysis') {
            steps {
                sh '''
                    mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.0.0.4389:sonar \
                    -Dsonar.projectKey=my-project \
                    -Dsonar.projectName=my-project \
                    -Dsonar.host.url=$SONAR_URL \
                    -Dsonar.token=$SONAR_TOKEN
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            --username "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Pull Image from Docker Hub') {
            steps {
                sh 'docker pull ${DOCKER_IMAGE}:${BUILD_NUMBER}'
            }
        }

        stage('Test Docker Container') {
            steps {
                sh '''
                    docker rm -f sonarproject-test || true

                    docker run -d \
                        --name sonarproject-test \
                        -p 8081:8080 \
                        ${DOCKER_IMAGE}:${BUILD_NUMBER}

                    sleep 15
                    docker ps --filter name=sonarproject-test
                    docker logs sonarproject-test

                    docker rm -f sonarproject-test
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Maven, SonarQube and Docker pipeline completed!'
        }

        failure {
            echo 'FAILED: Please check Jenkins Console Output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}