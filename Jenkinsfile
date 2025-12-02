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
        ARTIFACTORY_URL = 'http://13.127.91.182:8081'  // Replace with your EC2 IP
        DOCKER_REPO = 'docker-local'
        IMAGE_NAME = 'my-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
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
                // Get the full path of the SonarQubeScanner installation
                def scannerHome = tool name: 'SonarQubeScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                
                // Run sonar-scanner using full path
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

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh """
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ARTIFACTORY_URL}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${ARTIFACTORY_URL}/${DOCKER_REPO}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Login to JFrog') {
            steps {
                script {
                    echo "Logging into JFrog Artifactory..."
                    sh """
                        echo ${ARTIFACTORY_CREDS_PSW} | docker login ${ARTIFACTORY_URL} -u ${ARTIFACTORY_CREDS_USR} --password-stdin
                    """
                }
            }
        }
        
        stage('Push to JFrog Artifactory') {
            steps {
                script {
                    echo "Pushing Docker image to JFrog..."
                    sh """
                        docker push ${ARTIFACTORY_URL}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${ARTIFACTORY_URL}/${DOCKER_REPO}/${IMAGE_NAME}:latest
                    """
                    echo "✅ Image pushed successfully!"
                    echo "Image: ${ARTIFACTORY_URL}/${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
        stage('Build Artifact') {
            steps {
                echo 'Building Maven Artifact...'
                sh 'mvn clean package'
            }
        }

        stage('Docker Image Build') {
            steps {
                echo 'Building Docker Image...'
                sh "docker build -t jayesh7744/ultimate-cicd:${BUILD_NUMBER} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhub', variable: 'DOCKERHUB_TOKEN')]) {
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
            echo "Cleaning up workspace..."
            cleanWs()
        }
    }
}
