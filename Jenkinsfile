pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }

    tools {
        maven 'maven3'
    }

    

    environment {
        SONAR_URL = "http://65.1.100.50:9000"

        // JFrog Settings
        ARTIFACTORY_HOST = '13.127.91.182:8081'
        DOCKER_REPO = 'docker-local'
        IMAGE_NAME = 'my-app'
        IMAGE_TAG = "${BUILD_NUMBER}"

        // Credentials
        ARTIFACTORY_CREDS = credentials('jfrog-test')
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Cloning GitHub Repo...'
                git branch: 'main', url: 'https://github.com/mallinagraj/mergecheck.git'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                SONAR_TOKEN = credentials('sonarqube-token')
            }
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        def scannerHome = tool name: 'SonarQubeScanner',
                                               type: 'hudson.plugins.sonar.SonarRunnerInstallation'

                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=amrut \
                                -Dsonar.sources=./src \
                                -Dsonar.host.url=${SONAR_URL} \
                                -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage('Build Maven Artifact') {
            steps {
                echo 'Building Maven package...'
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image for JFrog') {
            steps {
                script {
                    echo "Building Docker image..."
                   sh """
    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

    docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
    13.127.91.182:8081/artifactory/docker-local/${IMAGE_NAME}:${IMAGE_TAG}

    docker tag ${IMAGE_NAME}:${IMAGE_TAG} \
    13.127.91.182:8081/artifactory/docker-local/${IMAGE_NAME}:latest

"""

                }
            }
        }

        stage('Login to JFrog Registry') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'jfrog-test',
                                               usernameVariable: 'USR',
                                               passwordVariable: 'PSW')]) {
                sh """
                    echo "$PSW" | docker login 13.127.91.182:8081/artifactory/docker-local -u "$USR" --password-stdin
                """
            }
        }
    }
}


                }
            }
        }

        stage('Push Images to JFrog') {
            steps {
                script {
                    sh """
    docker push 13.127.91.182:8081/artifactory/docker-local/${IMAGE_NAME}:${IMAGE_TAG}
    docker push 13.127.91.182:8081/artifactory/docker-local/${IMAGE_NAME}:latest

"""


                    echo "Image pushed:"
                    echo "${ARTIFACTORY_HOST}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Build DockerHub Image') {
            steps {
                echo "Building DockerHub Image..."
                sh "docker build -t jayesh7744/ultimate-cicd:${BUILD_NUMBER} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhub',
                        variable: 'DOCKERHUB_TOKEN')]) {

                        sh """
                            echo "${DOCKERHUB_TOKEN}" | docker login -u "jayesh7744" --password-stdin
                            docker push jayesh7744/ultimate-cicd:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
    }
}
